build_dir?=./build
NAME=$(shell yq r ./Kptfile 'openAPI.definitions."io.k8s.cli.setters.name".x-k8s-cli.setter.value')
# The kubectl context for your Kubeflow cluster
KFCTXT=$(NAME)

.PHONY: hydrate
hydrate:
	# Hydrate Common cert-manager
	rm -rf $(build_dir) && mkdir -p $(build_dir)
	mkdir -p $(build_dir)/cert-manager
	mkdir -p $(build_dir)/kubeflow-issuer
	kustomize build -o $(build_dir)/cert-manager ./
	kustomize build -o $(build_dir)/kubeflow-issuer ./cert-manager-1-3/cert-manager/kubeflow-issuer

.PHONY: apply
apply: hydrate
	# Apply Common cert-manager
# Try kpt live apply to simplify this steps.
	kubectl --context=$(KFCTXT) apply -f $(build_dir)/cert-manager/*v1_namespace_cert-manager.yaml
	kubectl --context=$(KFCTXT) apply -f $(build_dir)/cert-manager
	kubectl --context=$(KFCTXT) -n cert-manager wait --for=condition=Available --timeout=600s deploy cert-manager-webhook
	kubectl --context=$(KFCTXT) -n cert-manager wait --for=condition=Available --timeout=600s deploy cert-manager
	kubectl --context=$(KFCTXT) -n cert-manager wait --for=condition=Available --timeout=600s deploy cert-manager-cainjector

	# Common kubeflow-issuer
	kubectl --context=$(KFCTXT) apply -f $(build_dir)/kubeflow-issuer
