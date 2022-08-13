FROM node:14

WORKDIR /app

COPY package.json /app

RUN npm install

RUN npm install --global nodemon

COPY . /app

EXPOSE 80

ENV MONGODB_USERNAME=root
ENV MONGODB_PASSWORD=admin

CMD ["npm","start"]



