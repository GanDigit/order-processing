#### STAGE 1: Build ###
#
## We label our stage as ‘builder’
#FROM node:8.1.4-alpine as builder
#
#COPY ./ui/package.json ./ui/package-lock.json ./
#
### Storing node modules on a separate layer will prevent unnecessary npm installs at each build
#RUN npm i && mkdir /ng-app && cp -R ./node_modules ./ng-app
#
#WORKDIR /ng-app
#
#COPY . .
#
### Build the angular app in production mode and store the artifacts in dist folder
#RUN $(npm bin)/ng build --prod


### STAGE 2: Setup ###
FROM java:8

WORKDIR /

### From ‘builder’ stage copy over the artifacts in dist folder to default nginx public folder
#COPY --from=builder /ng-app/dist /src/main/resources/static

RUN sh ./gradlew build

ADD build/libs/orderprocessingweb-0.0.1-SNAPSHOT.jar ord_order_processing_web.jar

RUN sh -c 'touch /ord_order_processing_web.jar'

EXPOSE 80

ENTRYPOINT ["java","-jar","/ord_order_processing_web.jar"]
