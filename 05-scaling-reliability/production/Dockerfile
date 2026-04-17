# Build stage
FROM python:3.11-slim AS builder

WORKDIR /app
COPY 05-scaling-reliability/production/requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Final stage
FROM python:3.11-slim

WORKDIR /app
# Copy installed packages from builder
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# Copy source code and utils
COPY 05-scaling-reliability/production/app.py .
COPY 05-scaling-reliability/production/utils ./utils

# Metadata
ENV ENVIRONMENT=production
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
CMD ["python", "app.py"]
