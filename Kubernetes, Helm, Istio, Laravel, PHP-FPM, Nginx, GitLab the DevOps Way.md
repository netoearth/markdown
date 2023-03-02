04 Dec 2021[Kubernetes](https://nsirap.com/tags/kubernetes/)[DevOps](https://nsirap.com/tags/devops/)[Laravel](https://nsirap.com/tags/laravel/)[Istio](https://nsirap.com/tags/istio/)

I'll try to explain a full setup from the DevOps side of a full stack web application based on Laravel. Here is technologies that we will discuss here.

-   Kubernetes (GKE)
-   Cloud Repository (Google container repository service)
-   Istio
-   Helm
-   php-fpm
-   Nginx
-   Gitlab-ci
-   Laravel
-   VueJS

The application will be a single application with front and backend on the same git repository.

I made some updates on this post. This post is the big picture, and I go in some more details in the following related post.

-   [Kubernetes and Google Container Registry](https://nsirap.com/posts/041-kubernetes-and-google-cloud-registry/)
-   [Docker Best Practice, Multi-Stage Build](https://nsirap.com/posts/039-docker-best-practice-multi-stage-build/)
-   [Istio with Let's Encrypt Example](https://nsirap.com/posts/040-istio-with-lets-encrypt/)
-   [Gitlab Example with Laravel, Kebernetes and Helm](https://nsirap.com/posts/046-gitlab-example-with-laravel-kubernetes-and-helm/)
-   [x-forwarded-for with Istio and GCP Loadbalancer](https://nsirap.com/posts/047-x-forwarded-for-with-istio-kubernetes-and-gcp-load-balancer%20copy/)
-   [I made an update on ths part (Istio and Let's Encrypt) with a more detail approch](https://nsirap.com/posts/040-istio-with-lets-encrypt/)
-   [UPDATE: Gitlab CI/CD for Kubernetes, Helm and Laravel](https://nsirap.com/posts/053-gitlab-laravel-kubernetes-updated/)

## Nginx, PHP-FPM and a Laravel application

The approch here is each pod of a ReplicaSet will contains the following.

-   Envoy (Istio injection)
-   Nginx
-   PHP-FPM

This mean it will scale as much nginx/php-fpm that we have pods.  
The Nginx container and the PHP-FPM will need either JS/PNG/CSS or PHP.  
The PHP-FPM will be your container that contains everythings, the one you build on day to day basis with unit tests and everything. The Nginx container will receive the needed files on Init, will make a simple copy at start of the pod. This mean very low maintenance on the nginx container.

We need two parts, two `Dockerfile` for two containers.

### Laravel / php-fpm Dockerfile

```
FROM node:lts-alpine as node_buildWORKDIR /appCOPY package.json ./RUN npm installCOPY webpack.mix.js ./COPY resources/ ./resources/COPY public/ ./public/# fail du purge sinonRUN mkdir -p /public/cssRUN touch /public/css/app.cssRUN mkdir -p /public/jsRUN touch /public/js/app.jsRUN npm run prodFROM composer:2.1.9 as composer_build# voir pour le .lockCOPY ./composer.json /app/RUN composer install --no-dev --no-autoloader --no-scriptsCOPY . /appRUN composer install --no-dev --optimize-autoloaderFROM php:8.0-fpm-alpineRUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"RUN docker-php-ext-install pdo pdo_mysqlCOPY devops/docker/php/*.conf /usr/local/etc/php-fpm.d/COPY --chown=www-data --from=composer_build /app/ /var/www/html/COPY --from=node_build /app/public/ /var/www/html/public/RUN  php artisan view:cache# && php artisan route:cache \ fonctionne pas pour le moment
```

This is a multi-stage build of Docker

-   node, for building javascript assets
    -   Only copy the json part as it will have is own layer that could be cached if the package.json doesn't changes
    -   Some `mkdir` for internal purposes
    -   `npm` is run for production environment
-   composer, installation of vendors and autoloader
    -   The approch is the same, install vendors first will cache inside Docker.
-   Php-fpm alpine with basic setup

I add the following into the PHP configuration, to make php-fpm work.

A more in depth explaination is done here [Docker Best Practice, Multi-Stage Build](https://nsirap.com/posts/039-docker-best-practice-multi-stage-build/)

```
security.limit_extensions = php
```

### Nginx Dockerfile

I'll make some basic configuation inside a build.

```
FROM nginx:1.21.3-alpineCOPY vhost.conf /etc/nginx/conf.d/default.conf
```

With a vhost that contains this.

```
server {    listen       80;    listen  [::]:80;    server_name  localhost;    autoindex off;    gzip on;    index index.php;    client_max_body_size 500M;    # redirect server error pages to the static page /50x.html    #    error_page   500 502 503 504  /50x.html;    location = /50x.html {        root   /usr/share/nginx/html;    }    location ~* \.(jpg|jpeg|png|gif|ico|css)$ {        expires max;        add_header Cache-Control public;        access_log off;        try_files $uri $uri/ /index.php?$args;    }    location  ~ \.php$ {        fastcgi_pass   127.0.0.1:9000;        fastcgi_index  index.php;        fastcgi_param  SCRIPT_FILENAME /var/www/html/public/$fastcgi_script_name;        include        fastcgi_params;    }    location / {        try_files $uri $uri/ /index.php?$args;    }}
```

I does not have lots of experience with this setup, but this is what I came up with, and it works.

With this done, I have a Laravel up and running with php-fpm and nginx. You build both independently.

## Kubernetes, first approch

Let's build something that work on minikube, and lets make it simple, a single replica of 2 containers inside a pod. Let's call my-nginx, the customized nginx that make php calls to the other container build with Laravel, they both contains code, but it will be mounted on runtime.

-   my-nginx
-   my-app

```
apiVersion: apps/v1kind: Deploymentmetadata:  name: app  labels:    app: webspec:  replicas: 1  selector:    matchLabels:      app: web  template:    metadata:      labels:        app: web    spec:      containers:      - name: nginx        image: "my-nginx:1"        ports:        - containerPort: 80        volumeMounts:        - mountPath: /etc/nginx/html/          name: assets      - name: php        image: "my-app"        imagePullPolicy: IfNotPresent        ports:        - containerPort: 9000      initContainers:      - name: init-assets        image: "my-app"        imagePullPolicy: IfNotPresent        command: ['sh', '-c', "cp -r /var/www/html/public/* /assets/"]        volumeMounts:        - mountPath: /assets/          name: assets      volumes:      - name: assets        emptyDir: {}---apiVersion: v1kind: Servicemetadata:  name: my-servicespec:  type: NodePort  selector:    app: web  ports:      # By default and for convenience, the `targetPort` is set to the same value as the `port` field.    - port: 80      targetPort: 80
```

It's just a service on Port 80 (we'll see later the TLS support), it point on the nginx container that will send php request to `my-app`.

## Helm, Istio, Let's encrypt

This is a real life approch, let' have some tools working for us. It will be quite long as it contains lots of yaml. Let's do it.  
First, we setup the certificate issuers with let's encrypt if is not aleady done. This will make it work with istio.  
[I made an update on ths part (Istio and Let's Encrypt) with a more detail approch](https://nsirap.com/posts/040-istio-with-lets-encrypt/).

```
apiVersion: cert-manager.io/v1kind: ClusterIssuermetadata:  name: letsencrypt-prod  namespace: kube-systemspec:  acme:    # The ACME server URL    server: https://acme-v02.api.letsencrypt.org/directory    # Email address used for ACME registration    email: xxx@gmail.xxx    # Name of a secret used to store the ACME account private key    privateKeySecretRef:      name: letsencrypt-prod    # Enable the HTTP-01 challenge provider    solvers:    - http01:        ingress:          class: istio
```

Now, the istio gateway.

```
apiVersion: networking.istio.io/v1alpha3kind: Gatewaymetadata:  name: cluster-gatewayspec:  selector:    istio: ingressgateway # use istio default controller  servers:  - port:      number: 80      name: http      protocol: HTTP    hosts:    - my-website.com  - port:      number: 443      name: my-website      protocol: HTTPS    tls:      mode: SIMPLE      credentialName: "my-website-certs" # This should match the Certificate secretName    hosts:    - "my-website.com" # This should match a DNS name in the Certificate
```

Be sure to apply your gateway change on kubernetes to retreive the certificate right.  
You can check with this command.

```
kubectl get certificates -n istio-system
```

If something goes wrong, with `describe` can show with object to check for (order, challanges if I remember well).

### Helm

I am not showing every single file, juste the one I added for certificates or istio.  
Let's describe the `values.yaml`.

-   `env` will be loop in configMap (not sure it's the best way, but it was advised to me by better than me)
-   imagePullSecrets works with Google, I'll explain later in other post.
-   virtualService work with istio.
-   $DB\_PASSWORD\_PREPROD will be provide by a variable added to gitlab (more on that bellow)
-   $TAG is a native variable on gitlab.

```
# values.yamlreplicaCount: 1namespace: defaultimage:  repository: xxxxx  pullPolicy: Always  # Overrides the image tag whose default is the chart appVersion.  tag: $TAGimagePullSecrets:- name: gcr-ionameOverride: ""fullnameOverride: ""serviceAccount:  # Specifies whether a service account should be created  create: true  # Annotations to add to the service account  annotations: {}  # The name of the service account to use.  # If not set and create is true, a name is generated using the fullname template  name: ""podAnnotations:  "cluster-autoscaler.kubernetes.io/safe-to-evict": "true"  "sidecar.istio.io/proxyCPU": "20m"virtualService:  url: "xxx.xxx.fr"podSecurityContext: {}  # fsGroup: 2000securityContext: {}  # capabilities:  #   drop:  #   - ALL  # readOnlyRootFilesystem: true  # runAsNonRoot: true  # runAsUser: 1000service:  type: ClusterIP  port: 80resources:  limits:    cpu: 300m    memory: 512Mi  requests:    cpu: 20m    memory: 128Mienv:  LOG_CHANNEL: stderr  APP_ENV: local # FIXME  APP_NAME: xxx  APP_KEY: base64:xxx=  APP_DEBUG: true # FIXME  APP_URL: https://xxx.fr  DB_CONNECTION: mysql  DB_HOST: "xxx"  DB_DATABASE: xxx  DB_USERNAME: xxx  DB_PASSWORD: $DB_PASSWORD_PREPRODautoscaling:  enabled: false  minReplicas: 1  maxReplicas: 100  targetCPUUtilizationPercentage: 80  # targetMemoryUtilizationPercentage: 80nodeSelector: {}tolerations: []affinity: {}
```

If you wender where the `ImagePullSecret gcr-io` come from, I made a post where more details on this. [Kubernetes and Google Container Registry](https://nsirap.com/posts/041-kubernetes-and-google-cloud-registry/)

Now, for the template part of Helm

First, a certificates that works with let's encript and istio.

```
# certificates.yamlapiVersion: cert-manager.io/v1kind: Certificatemetadata:  name: {{ include "xxx.fullname" . }}-certs  namespace: istio-systemspec:  secretName: {{ include "xxx.fullname" . }}-certs  issuerRef:    name: letsencrypt-prod    kind: ClusterIssuer  commonName: {{ .Values.virtualService.url }}  dnsNames:  - {{ .Values.virtualService.url }}
```

Now, a simple loop for the `configMap.yaml`, with a simple key / value iteration.

```
apiVersion: v1kind: ConfigMapmetadata:  name: {{ include "nci-authentication.fullname" . }}-env  namespace: {{ .Values.namespace }}data:  {{- range $key, $val := .Values.env }}    {{ $key }}: {{ $val | quote }}  {{- end }}
```

I guess I have to copy/paste my deployment.yaml as it contains the initialization part, and the configMap. I know it will be verbose, and I apologies for this.

```
apiVersion: apps/v1kind: Deploymentmetadata:  name: {{ include "xxx.fullname" . }}  namespace: {{ .Values.namespace }}  labels:    {{- include "xxx.labels" . | nindent 4 }}spec:  {{- if not .Values.autoscaling.enabled }}  replicas: {{ .Values.replicaCount }}  {{- end }}  selector:    matchLabels:      {{- include "xxx.selectorLabels" . | nindent 6 }}  template:    metadata:      {{- with .Values.podAnnotations }}      annotations:        rollme: {{ randAlphaNum 5 | quote }} # force le roll des pods, surprennant, mais dans la doc officielle        {{- toYaml . | nindent 8 }}      {{- end }}      labels:        {{- include "xxx.selectorLabels" . | nindent 8 }}    spec:      {{- with .Values.imagePullSecrets }}      imagePullSecrets:        {{- toYaml . | nindent 8 }}      {{- end }}      serviceAccountName: {{ include "xxx.serviceAccountName" . }}      securityContext:        {{- toYaml .Values.podSecurityContext | nindent 8 }}      containers:        - name: {{ .Chart.Name }}          image: "my-nginx:1"          imagePullPolicy: Always          ports:            - name: http              containerPort: 80              protocol: TCP          volumeMounts:          - mountPath: /etc/nginx/html/            name: assets        - name: {{ .Chart.Name }}-backend          securityContext:            {{- toYaml .Values.securityContext | nindent 12 }}          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"          imagePullPolicy: {{ .Values.image.pullPolicy }}          ports:            - name: fpm              containerPort: 9000              protocol: TCP#          livenessProbe:#            httpGet:#              path: /#              port: http#          readinessProbe:#            httpGet:#              path: /#              port: http          resources:            {{- toYaml .Values.resources | nindent 12 }}          env:          {{- $dot := . }}          {{- range $key, $val := .Values.env }}          - name: {{ $key }}            valueFrom:              configMapKeyRef:                name: {{ include "xxx.fullname" $dot }}-env                key: {{ $key }}          {{- end }}      initContainers:      - name: init-assets        image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"        imagePullPolicy: IfNotPresent        command: ['sh', '-c', "cp -r /var/www/html/public/* /assets/"]        volumeMounts:        - mountPath: /assets/          name: assets      volumes:      - name: assets        emptyDir: {}      {{- with .Values.nodeSelector }}      nodeSelector:        {{- toYaml . | nindent 8 }}      {{- end }}      {{- with .Values.affinity }}      affinity:        {{- toYaml . | nindent 8 }}      {{- end }}      {{- with .Values.tolerations }}      tolerations:        {{- toYaml . | nindent 8 }}      {{- end }}
```

This is the same approch as earlier, with a copy of code in the `InitContainers`. I'm not sure it is the best approch, but this is a low maintenance approch with absolutly 100% certenty to run the right code on each node.

I'm not showing the `hpa.yaml`, and the `service.yaml` is not very interreseting.

```
apiVersion: v1kind: Servicemetadata:  name: {{ include "xxx.fullname" . }}  namespace: {{ .Values.namespace }}  labels:    {{- include "xxx.labels" . | nindent 4 }}spec:  type: {{ .Values.service.type }}  ports:    - port: {{ .Values.service.port }}      targetPort: http      protocol: TCP      name: http  selector:    {{- include "xxx.selectorLabels" . | nindent 4 }}
```

Last, the `virtualservice.yaml` is specific to the istio part.

```
apiVersion: networking.istio.io/v1alpha3kind: VirtualServicemetadata:  name: {{  include "xxx.fullname" . }}spec:  hosts:  - {{ .Values.virtualService.url }}  gateways:  - cluster-gateway  http:  - match:    - uri:        prefix: "/"    route:    - destination:        host: {{  include "xxx.fullname" . }}.{{ .Values.namespace }}.svc.cluster.local        port:          number: 80
```

I wont explain helm commands to install or upgrade releases, this is not a beginner tutorial on how to deploy with helm.

Now, this is running on a Kubernetes cluster, with Istio.

## Gitlab

Let's just add a runner to this, it's not yet complete but I'll add an update soon.

Update: a more in depht post as been added : [Gitlab Example with Laravel, Kebernetes and Helm](https://nsirap.com/posts/046-gitlab-example-with-laravel-kubernetes-and-helm/)

`.gitlab-ci.yaml`

```
 variables:  IMAGE: eu.gcr.io/xxxx/xxxx  TAG: "${CI_COMMIT_SHORT_SHA}"  CONNECT_K8S_PREPROD_CMD: gcloud container clusters get-credentials kube-xxxx --zone europe-west1-b --project xxxxstages:  - build  - test  - maintenance_down  - migration  - maintenance_up  - delivery_stagingbuild:  stage: build  tags:    - gcp-shell  script:    - docker build -t ${IMAGE}:${TAG} .    - gcloud auth activate-service-account gitlab-push-container@xxxx.gserviceaccount.com --key-file=/gitlab.json    - gcloud auth configure-docker    - docker push ${IMAGE}:${TAG}test:  stage: test  image: ${IMAGE}:${TAG}  tags:    - gcp-docker  before_script:    - curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer    - cd /var/www/html    - composer install # on a besoin des dépendences de dev (contrairement à la prod)    - apk add gettext # envsubst    - envsubst < /var/www/html/devops/.env.test > /var/www/html/.env    - php artisan key:generate  script:    - ./vendor/phpunit/phpunit/phpunitmaintenance_down:  stage: maintenance_down  tags:    - gcp-shell  script:    - echo "MAINTENANCE=DOWN" >> build.env    - kubectl get po -l app.kubernetes.io/name=my-app-backend -o name | xargs -I{} kubectl exec {} -c my-app-backend -- php artisan down  artifacts:    reports:      dotenv: build.envmigration:  stage: migration  image: ${IMAGE}:${TAG}  tags:    - gcp-docker  before_script:    - apk add gettext # envsubst    - envsubst < /var/www/html/devops/.env.test > /var/www/html/.env    - cd /var/www/html    - php artisan key:generate  script:    - cd /var/www/html    - php artisan migratemaintenance_up:  stage: maintenance_up  dependencies:    - maintenance_down  tags:    - gcp-shell  when: always  script:    - echo $MAINTENANCE    - ${CONNECT_K8S_PREPROD_CMD}    - (if [ "$MAINTENANCE" == "DOWN" ]; then kubectl get po -l app.kubernetes.io/name=my-app -o name | xargs -I{} kubectl exec {} -c my-app-backend -- php artisan up; fi);delivery_staging:  stage: delivery_staging  tags:    - gcp-shell  script:    - VALUES_FILE=devops/helm/stage.yaml    - ${CONNECT_K8S_PREPROD_CMD}    - envsubst < ${VALUES_FILE} > devops/helm/dist-stage.yaml    - helm upgrade -f devops/helm/dist-stage.yaml xxxxx devops/helm/
```

There is a lot going on in this file, i'll explain in more details soon in a post. I'll give some update here soon.  
Update: a more in depht post as been added : [Gitlab Example with Laravel, Kebernetes and Helm](https://nsirap.com/posts/046-gitlab-example-with-laravel-kubernetes-and-helm/)

## Conclusion

This is a full setup of an application with Kubernetes, Istio (i'm not naming every technologies again...). I'm not going to deep on explainations, but I'll give some update soon.

I hope it could help someone, as it took me some time to summarize all this.

___

-   Next: [Docker Best Practice, Multi-Stage Build](https://nsirap.com/posts/039-docker-best-practice-multi-stage-build/)
-   Previous: [6 months of DevOps mentoring](https://nsirap.com/posts/037-6-months-of-devops-mentoring/)