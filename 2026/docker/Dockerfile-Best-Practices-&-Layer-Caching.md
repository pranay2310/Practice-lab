Learn how to write efficient Dockerfiles by understanding layer caching, minimizing image size, and applying best practices. You will build real Docker images inside a Docker-in-Docker environment, compare naive vs optimized Dockerfiles, and observe how layer caching works in

--------------------------------------------------------------------------
Create a Naive Dockerfile
Create a working directory and write a naive Dockerfile that copies all files before installing dependencies. This is a common anti-pattern that busts the cache every time source code changes.

1. Create a project directory:
mkdir -p /lab/naive && cd /lab/naive

2. Create a simple requirements file:
echo 'flask==2.3.2' > requirements.txt

3. Create a dummy app file:
echo 'print("hello")' > app.py

4. Write the naive Dockerfile (copies everything first, then installs):
cat > Dockerfile <<'EOF'
FROM python:3.11-alpine
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python", "app.py"]
EOF


5. Build the image and note the time:
docker build -t naive-app:v1 .


6. Make a small change to app.py and rebuild to observe cache busting:
echo 'print("hello again")' > app.py
docker build -t naive-app:v2 .
Show hint
Verify Task
Show verify command
Hide solution
SOLUTION

Copy
mkdir -p /lab/naive && cd /lab/naive
echo 'flask==2.3.2' > requirements.txt
echo 'print("hello")' > app.py
cat > Dockerfile <<'EOF'
FROM python:3.11-alpine
WORKDIR /app
COPY . .
RUN pip install --no-cache-dir -r requirements.txt
CMD ["python", "app.py"]
EOF
docker build -t naive-app:v1 .
echo 'print("hello again")' > app.py
docker build -t naive-app:v2 .

------------------------------------------------------------------------------
TASK 2 OF 5
Done
25 pts
Create an Optimized Dockerfile with Proper Layer Ordering
Now create an optimized Dockerfile that copies the dependency file first, installs dependencies, and only then copies the rest of the source code. This ensures the pip install layer is cached as long as requirements.txt does not change.

1. Create a new directory:
mkdir -p /lab/optimized && cd /lab/optimized

2. Copy the same files:
echo 'flask==2.3.2' > requirements.txt
echo 'print("hello")' > app.py

3. Write the optimized Dockerfile:
cat > Dockerfile <<'EOF'
FROM python:3.11-alpine
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
EOF

4. Build the first version:
docker build -t optimized-app:v1 .

5. Change only app.py and rebuild — notice pip install is cached:
echo 'print("hello again")' > app.py
docker build -t optimized-app:v2 .
Show hint
✓ Verified
Show verify command
✓ Task passed!

optimized-app:v1
optimized-app:v2
Hide solution
SOLUTION

Copy
mkdir -p /lab/optimized && cd /lab/optimized
echo 'flask==2.3.2' > requirements.txt
echo 'print("hello")' > app.py
cat > Dockerfile <<'EOF'
FROM python:3.11-alpine
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
EOF
docker build -t optimized-app:v1 .
echo 'print("hello again")' > app.py
docker build -t optimized-app:v2 .

-----------------------------------------------------------------------------