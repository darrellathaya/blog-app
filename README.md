# 1. Project Structure
    /blog_app
        │
        ├── app.py
        ├── instance/
        │   └── blog.db
        ├── templates/
        │   └── index.html
        |   └── create.html
        └── Dockerfile

# 2. Flask-App Container
## Install Dependencies
    sudo dnf install -y podman podman-compose python3 python3-pip
    
## Set Up the Project Directory
    mkdir -p blog-app
    cd blog-app

## Create the Dockerfile
    # Use an official Python runtime as a parent image
    FROM python:3.10-slim

    # Set the working directory in the container
    WORKDIR /app

    # Copy the current directory contents into the container
    COPY . /app

    # Install dependencies
    RUN pip install --no-cache-dir -r requirements.txt

    # Expose the Flask app port
    EXPOSE 5000

    # Command to run the Flask app
    CMD ["python", "app.py"]


## Create app.py
    from flask import Flask, render_template, request, redirect, url_for
    import psycopg2

    app = Flask(__name__)

    # Database connection settings
    DB_HOST = "your_db_host"
    DB_NAME = "your_db_name"
    DB_USER = "your_db_user"
    DB_PASSWORD = "your_db_password"

    # Database initialization
    def init_db():
        conn = psycopg2.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASSWORD
        )
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS posts (
                id SERIAL PRIMARY KEY,
                title TEXT NOT NULL,
                content TEXT NOT NULL
            )
        ''')
        conn.commit()
        conn.close()

    @app.route('/')
    def index():
        conn = psycopg2.connect(
            host=DB_HOST,
            database=DB_NAME,
            user=DB_USER,
            password=DB_PASSWORD
        )
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM posts ORDER BY id DESC")
        posts = cursor.fetchall()
        conn.close()
        return render_template('index.html', posts=posts)

    @app.route('/add', methods=['GET', 'POST'])
    def add_post():
        if request.method == 'POST':
            title = request.form['title']
            content = request.form['content']
            conn = psycopg2.connect(
                host=DB_HOST,
                database=DB_NAME,
                user=DB_USER,
                password=DB_PASSWORD
            )
            cursor = conn.cursor()
            cursor.execute("INSERT INTO posts (title, content) VALUES (%s, %s)", (title, content))
            conn.commit()
            conn.close()
            return redirect(url_for('index'))
        return render_template('add_post.html')

    if __name__ == "__main__":
        init_db()
        app.run(debug=True, host='0.0.0.0')
