FROM node:16-alpine

ENV NODE_ENV=production \
    PORT=80
WORKDIR /usr/src/app

COPY --chown=node:node . .
RUN npm clean-install --omit=dev
USER node
EXPOSE $PORT
CMD ["npm", "start"]
