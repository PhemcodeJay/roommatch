# RoomMatch – roommate, room & sublet listings (Flask + SQLite)

## Run locally
    pip install -r requirements.txt
    python app.py            # http://localhost:5000

The SQLite database (`roommatch.db`) and a session signing key (`.secret_key`) are created on first run.

## Deploy to Render

### Using Gunicorn (Recommended)

Set these environment variables in the Render dashboard:

| Variable | Value |
|---|---|
| `SECRET_KEY` | `<long random string>` (keep this secret!) |
| `COOKIE_SECURE` | `1` (set when behind HTTPS) |
| `DB_PATH` | `/data/roommatch.db` (use Render's persistent disk) |
| `TWILIO_ACCOUNT_SID` | Your Twilio account SID (optional, for WhatsApp) |
| `TWILIO_AUTH_TOKEN` | Your Twilio auth token (optional) |
| `TWILIO_WHATSAPP_NUMBER` | Your Twilio WhatsApp number (optional) |
| `PORT` | `10000` (Render's default port) |

**Render Settings:**
- Build Command: `pip install -r requirements.txt`
- Start Command: `gunicorn -w 2 -b 0.0.0.0:$PORT app:app`
- Add the environment variables listed above under "Environment Variables"

### Using Docker (Alternative)

If you prefer Docker, create a `Dockerfile`:

```dockerfile
FROM python:3.14-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

EXPOSE 10000

ENV PYTHONUNBUFFERED=1
ENV SECRET_KEY=<your-secret-key>
ENV COOKIE_SECURE=1
ENV DB_PATH=/data/roommatch.db

CMD ["gunicorn", "-w", "2", "-b", "0.0.0.0:$PORT", "app:app"]
```

### Important Deployment Notes

1. **Database Persistence**: On Render, add a persistent disk at `/data/` and set `DB_PATH=/data/roommatch.db`. Without this, the database resets on every redeploy.

2. **Cookie Security**: Set `COOKIE_SECURE=1` when serving over HTTPS. This ensures session cookies are only sent over secure connections.

3. **Secret Key**: Generate a strong random string for `SECRET_KEY`. Keep this secret - it's used to sign session cookies.

4. **Port**: Render assigns a port via the `$PORT` environment variable (default 10000). The app reads this from `os.environ.get("PORT", 5000)`.

5. **WhalsApp Configuration**: To enable WhatsApp features, set:
   - `TWILIO_ACCOUNT_SID`
   - `TWILIO_AUTH_TOKEN`
   - `TWILIO_WHATSAPP_NUMBER`

6. **Static Files**: The app serves `static/index.html` at the root path. Ensure your static files are in the `static/` directory.

7. **Process Model**: The app uses `gunicorn` with 2 worker processes. Render sets `WEB_CONCURRENCY` automatically.

## Deploy to Other Platforms

The app can also be deployed to Fly.io, Railway, Heroku, or any platform that supports Python/Flask apps. The key requirements are:

- Set `SECRET_KEY`, `COOKIE_SECURE`, and `DB_PATH` environment variables
- Use gunicorn or waitress to serve the app
- Set the PORT environment variable to match the platform's convention
- Ensure the database directory is writable and persistent

## Local Development

```bash
pip install -r requirements.txt
python app.py            # http://localhost:5000
```

Or with gunicorn:
```bash
gunicorn -w 2 -b 0.0.0.0:5000 app:app
```

## Environment Variables Reference

| Variable | Required | Default | Description |
|---|---|---|---|
| `SECRET_KEY` | Yes | Auto-generated | Session signing key |
| `COOKIE_SECURE` | No | `0` | Set to `1` when behind HTTPS |
| `DB_PATH` | No | `roommatch.db` | Path to SQLite database |
| `TWILIO_ACCOUNT_SID` | No | None | Twilio account SID |
| `TWILIO_AUTH_TOKEN` | No | None | Twilio auth token |
| `TWILIO_WHATSAPP_NUMBER` | No | `whatsapp:+14155238886` | Twilio WhatsApp number |
| `PORT` | No | `5000` | Port to listen on |

## Security Notes

- Passwords are hashed (scrypt via Werkzeug)
- Sessions use signed HttpOnly SameSite=Lax cookies
- Writes require JSON (blocks cross-site form posts)
- All SQL is parameterized
- Login is rate limited (10 failures / 15 min per IP or email)
- Content-Security-Policy header is set by default

Not included: email verification and password reset (these need an email service).
