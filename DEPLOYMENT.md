# Deployment Guide for Render

This guide will help you deploy your Flask blog application to Render.

## Prerequisites

1. A GitHub account
2. A Render account (sign up at https://render.com)
3. Your code pushed to a GitHub repository

## Step-by-Step Deployment

### 1. Push Your Code to GitHub

If you haven't already, push your code to a GitHub repository:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main
```

### 2. Create a PostgreSQL Database on Render

1. Go to your Render dashboard
2. Click "New +" → "PostgreSQL"
3. Configure:
   - **Name**: `blogdb` (or any name you prefer)
   - **Database**: `blogdb`
   - **User**: `blogdb`
   - **Region**: Choose closest to you
   - **PostgreSQL Version**: Latest
   - **Plan**: Free tier is fine for development
4. Click "Create Database"
5. **Important**: Copy the **Internal Database URL** (you'll need this later)

### 3. Deploy the Web Service

#### Option A: Using render.yaml (Recommended)

1. Go to Render dashboard
2. Click "New +" → "Blueprint"
3. Connect your GitHub repository
4. Render will automatically detect `render.yaml` and configure everything
5. Review the settings and click "Apply"

#### Option B: Manual Setup

1. Go to Render dashboard
2. Click "New +" → "Web Service"
3. Connect your GitHub repository
4. Configure the service:
   - **Name**: `blog-with-users` (or any name)
   - **Environment**: `Python 3`
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn main:app`
5. Add Environment Variables:
   - `SECRET_KEY`: Generate a strong random key (you can use: `python -c "import secrets; print(secrets.token_hex(32))"`)
   - `DATABASE_URL`: Paste the **Internal Database URL** from step 2
   - `FLASK_ENV`: `production`
6. Click "Create Web Service"

### 4. Link the Database

If you used manual setup, you need to link the database:

1. In your Web Service settings, go to "Environment"
2. Click "Link Database" or manually add:
   - Key: `DATABASE_URL`
   - Value: The Internal Database URL from your PostgreSQL database

### 5. Initialize the Database

After deployment, you need to create the database tables:

1. Go to your Web Service on Render
2. Click "Shell" tab
3. Run:
   ```bash
   python
   ```
4. Then in Python:
   ```python
   from main import app, db
   with app.app_context():
       db.create_all()
   ```
5. Exit Python: `exit()`

### 6. Create Your Admin User

1. In the Shell, run:
   ```python
   from main import app, db, User
   from werkzeug.security import generate_password_hash
   with app.app_context():
       admin = User(
           name="Your Name",
           email="your-email@example.com",
           password=generate_password_hash("your-secure-password")
       )
       db.session.add(admin)
       db.session.commit()
   ```

**Note**: The first user you create will have `id=1`, which is the admin user.

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| `SECRET_KEY` | Flask secret key for sessions | Yes |
| `DATABASE_URL` | PostgreSQL connection string | Yes (auto-linked if using Blueprint) |
| `FLASK_ENV` | Set to `production` | Recommended |

## Important Notes

1. **File Uploads**: Files uploaded to `static/uploads/` are stored on the filesystem, which is **ephemeral** on Render. Files may be lost when the service restarts. For production, consider using cloud storage (AWS S3, Cloudinary, etc.).

2. **Database**: The free tier PostgreSQL database on Render will be deleted after 90 days of inactivity. Consider upgrading for production use.

3. **HTTPS**: Render automatically provides HTTPS for your application.

4. **Custom Domain**: You can add a custom domain in the service settings.

## Troubleshooting

### Application won't start
- Check the logs in Render dashboard
- Verify all environment variables are set
- Ensure `requirements.txt` includes all dependencies

### Database connection errors
- Verify `DATABASE_URL` is set correctly
- Make sure you're using the **Internal Database URL** (not External)
- Check that the database is linked to your web service

### 500 errors
- Check application logs
- Verify database tables are created
- Ensure admin user exists

## Updating Your Application

1. Push changes to your GitHub repository
2. Render will automatically detect and deploy the changes
3. Monitor the deployment in the Render dashboard

