PROJECT=orderproc2
DOCKER_REPOSITORY=pc310-1.icp:8500
DOCKER_REPOSITORY_REMOTE=pc310-1.icp:8500

PROXY_NODE=9.204.168.81
PROXY=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
UIPROXY=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
URL_SERVICE_CATALOG=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)
URL_SERVICE_ORDER=http://$(PROXY_NODE):$(shell kubectl get svc istio-ingressgateway -n istio-system -o json | jq .spec.ports[0].nodePort)


DOCKER_REGISTRY_CATALOG=$(DOCKER_REPOSITORY)/$(PROJECT)/catalog
DOCKER_REGISTRY_ORDER=$(DOCKER_REPOSITORY)/$(PROJECT)/order
DOCKER_REGISTRY_WEB_CLIENT=$(DOCKER_REPOSITORY)/$(PROJECT)/webclient
DOCKER_REGISTRY_UI=$(DOCKER_REPOSITORY)/$(PROJECT)/ui
DOCKER_REGISTRY_COMPOSIT_APP=$(DOCKER_REPOSITORY)/$(PROJECT)/compositeop

FORTIO_POD=$(shell kubectl get pod -n $(PROJECT) -l app=fortio -o jsonpath='{.items[0].metadata.name}')
DEBUG_POD=$(shell kubectl get pod -n $(PROJECT) -l app=curl -o jsonpath='{.items[0].metadata.name}')

init:
	-bash -c 'kubectl create ns $(PROJECT)'
	-bash -c 'kubectl label namespace $(PROJECT) istio-injection=enabled'
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./imagepolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./debug.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./psp.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./ingress.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./config/fortio-deploy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./destinationrules.yaml | kubectl apply -f -

clean:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./destinationrules.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./messaging/redis.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./debug.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./config/fortio-deploy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./ingress.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./psp.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./imagepolicy.yaml | kubectl delete -f -
	-bash -c 'kubectl delete ns $(PROJECT)'


init-app-catalog:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/service.yaml | kubectl apply -f -

build-app-catalog:
	docker build -t $(DOCKER_REGISTRY_CATALOG):v1.0.0 ./../catalog-service

push-app-catalog:
	docker push $(DOCKER_REGISTRY_CATALOG):v1.0.0
	
deploy-app-catalog: build-app-catalog push-app-catalog
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_CATALOG):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/deployment.yaml  | kubectl apply -f -

clean-app-catalog:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_CATALOG):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_REGISTRY_CATALOG):v1.0.0
	-docker rm $(DOCKER_REGISTRY_CATALOG):v1.0.0
	-docker rmi $(DOCKER_REGISTRY_CATALOG):v1.0.0

test-app-catalog:
	-bash -c './../catalog-service/config/test.sh $(PROXY)'


init-app-order:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/service.yaml | kubectl apply -f -

build-app-order:
	docker build -t $(DOCKER_REGISTRY_ORDER):v1.0.0 ./../order-service

push-app-order:
	docker push $(DOCKER_REGISTRY_ORDER):v1.0.0

deploy-app-order: build-app-order push-app-order
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_ORDER):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/deployment.yaml  | kubectl apply -f -

clean-app-order:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_ORDER):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_REGISTRY_ORDER):v1.0.0
	-docker rm $(DOCKER_REGISTRY_ORDER):v1.0.0
	-docker rmi $(DOCKER_REGISTRY_ORDER):v1.0.0

test-app-order:
	-bash -c './../order-service/config/test.sh $(PROXY)'

init-app-webclient:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/service.yaml | kubectl apply -f -
	-sed -e 's~<CATALOG_SERVICE_URL>~$(URL_SERVICE_CATALOG)~g' ./../webclient/ui/src/environments/environment.ts.template > ./../webclient/ui/src/environments/environment.ts.template
    -sed -e 's~<ORDER_SERVICE_URL>~$(ORDER_SERVICE_URL)~g' ./../webclient/ui/src/environments/environment.ts.template > ./../webclient/ui/src/environments/environment.ts
    -sed -e 's~<CATALOG_SERVICE_URL>~$(URL_SERVICE_CATALOG)~g' ./../webclient./ui/src/environments/environment.prod.ts.template > ./../webclient/ui/src/environments/environment.prod.ts.template
    -sed -e 's~<ORDER_SERVICE_URL>~$(ORDER_SERVICE_URL)~g' ./../webclient/ui/src/environments/environment.prod.ts.template > ./../webclient/ui/src/environments/environment.prod.ts

build-app-webclient:
	docker build -t $(DOCKER_REGISTRY_WEBCLIENT):v1.0.0 ./../webclient

push-app-webclient:
	docker push $(DOCKER_REGISTRY_WEBCLIENT):v1.0.0

deploy-app-webclient: build-app-webclient push-app-webclient
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_WEBCLIENT):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/deployment.yaml  | kubectl apply -f -

clean-app-webclient:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/service.yaml | kubectl delete -f -
	-sed -e 's~<CONTAINER>~$(DOCKER_REGISTRY_WEBCLIENT):v1.0.0~g' -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/deployment.yaml | kubectl delete -f -

	-docker stop $(DOCKER_REGISTRY_WEBCLIENT):v1.0.0
	-docker rm $(DOCKER_REGISTRY_WEBCLIENT):v1.0.0
	-docker rmi $(DOCKER_REGISTRY_WEBCLIENT):v1.0.0

test-app-webclient:
	-bash -c './../webclient/config/test.sh $(PROXY)'

init-all: init-app-catalog init-app-order init-app-webclient
build-all: build-app-catalog build-app-order build-app-webclient
push-all: push-app-catalog push-app-order push-app-webclient
deploy-all: deploy-app-catalog deploy-app-order deploy-app-webclient
clean-all: clean-app-catalog clean-app-order clean-app-webclient
test-all: test-app-catalog test-app-order test-app-webclient

initnp:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/networkpolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/networkpolicy.yaml | kubectl apply -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/networkpolicy.yaml | kubectl apply -f -

cleannp:
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../catalog-service/config/networkpolicy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../order-service/config/networkpolicy.yaml | kubectl delete -f -
	-sed -e 's~<PROJECT>~$(PROJECT)~g' ./../webclient/config/networkpolicy.yaml | kubectl delete -f -


WEB_CLIENT_POD_NAME=$(shell kubectl -n $(PROJECT) get pod -l app=$(PROJECT)-webclient-service -o jsonpath='{.items[0].metadata.name}')

WEB_CLIENT_access:
	$(shell kubectl -n $(PROJECT) port-forward $(WEB_CLIENT_POD_NAME) 9080:80)

restart-all:
	kubectl delete pods --all -n $(PROJECT)