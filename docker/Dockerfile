FROM python:3-alpine as test
ADD ./ ./
RUN pip3 install --no-cache-dir -e ".[dev]"
RUN python -m pytest ./test 

FROM certbot/certbot
ADD ./ ./
RUN chmod +x strato_certbot/*.py
RUN pip3 install --no-cache-dir -e .
ENTRYPOINT ["sh",  "-c", "certbot certonly --agree-tos --no-eff-email --email $EMAIL --manual --preferred-challenges dns --manual-auth-hook /opt/certbot/auth-hook.py --manual-cleanup-hook /opt/certbot/cleanup-hook.py  -d $DOMAIN -d *.$DOMAIN"]
