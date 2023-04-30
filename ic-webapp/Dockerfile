FROM python:3.6-alpine
WORKDIR /opt/
RUN apk update
RUN apk add git
RUN pip install flask==1.1.2
RUN git clone https://github.com/sadofrazer/ic-webapp.git /opt/
EXPOSE 8080
ENV ODOO_URL="https://www.odoo.com"
ENV PGADMIN_URL="https://www.pgadmin.org"
ENTRYPOINT ["python", "app.py"]