FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
#RUN npm install
EXPOSE ${PORT}
#CMD ["npm", "start"]

# This will copy startup script and make it executable
COPY start.sh /start.sh
RUN chmod +x /start.sh

# This will call the external script
ENTRYPOINT ["/start.sh"]