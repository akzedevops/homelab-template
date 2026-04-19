FROM python:3.12-slim AS build
WORKDIR /app
COPY mkdocs.yml .
COPY docs/ docs/
RUN pip install --no-cache-dir mkdocs-material && mkdocs build

FROM nginx:alpine
COPY --from=build /app/site /usr/share/nginx/html
EXPOSE 80
