# securethebox-challenge

# Tips
- User must know some basics of using Linux OS
- bash/sh, vi, ps, netstat
- Basic knowledge of what a WAF does
- What are regular expressions
- SQL syntax
- OWASP top 10

# Requirements for Local Development
- Edit your local dns configuration file (/etc/hosts for mac)
```
127.0.0.1       monitor.securethebox.us
127.0.0.1       splunk.securethebox.us
127.0.0.1       waf.securethebox.us
127.0.0.1       waf-editor.securethebox.us
127.0.0.1       app.securethebox.us
127.0.0.1       editor-app.securethebox.us
```
- Install npm
- Install docker
- Install required packages:
```
cd securethebox-challenge
npm install
```
- Run Challenge:
```
npm run challenge
```
- View Challange:
```
visit waf.securethebox.us to view the App behind ModSecurity
visit waf-editor.securethebox.us to use CloudCmd to edit the files of modsecurity
visit app.securethebox.us to view the app without the firewall
visit editor-app.securethebox.us to use CloudCmd to edit the files of vulnerable app
```

# Notes
**gke**
```
gcloud auth login
gcloud config set project "securethebox"
gcloud config set compute/region "us-central1"
gcloud config set compute/zone "us-central1-a"
gcloud container clusters create "external-dns" \
    --num-nodes 1 \
    --scopes "https://www.googleapis.com/auth/ndev.clouddns.readwrite"
gcloud container clusters get-credentials external-dns --zone us-central1-a --project securethebox
gcloud dns managed-zones create "external-dns-test-securethebox-us" \
    --dns-name "external-dns-test.securethebox.us." \
    --description "Automatically managed zone by kubernetes.io/external-dns"
gcloud dns record-sets list \
    --zone "external-dns-test-securethebox-us" \
    --name "external-dns-test.securethebox.us." \
    --type NS
gcloud dns record-sets transaction start --zone "gcp-zalan-do"
gcloud dns record-sets transaction add ns-cloud-e{1..4}.googledomains.com. \
    --name "external-dns-test.securethebox.us." --ttl 300 --type NS --zone "securethebox-us"
gcloud dns record-sets transaction execute --zone "securethebox-us"
```

**Heroku - Getting Started**
<https://devcenter.heroku.com/articles/container-registry-and-runtime>
```
brew tap heroku/brew && brew install heroku
heroku container:login

heroku create stb-traefik
docker build -t stb-traefik .
docker tag stb-traefik jidokaus/stb-traefik:latest
heroku container:push web
heroku container:release web


"heroku-deploy": "npm run docker-build && npm run heroku-replace ; npm run docker-tag ; npm run dockerhub-push",
"heroku-replace": "heroku apps:destroy --app $npm_package_config_servername --confirm $npm_package_config_servername ; git init && heroku create $npm_package_config_servername && heroku container:push web ; heroku logs",
"set-domain": "heroku config:set REDIRECT_DOMAIN=$npm_package_config_herokusite",
"docker-build": "npm run build-client && docker build -t traefik .",
"docker-clean": "docker stop $(docker ps -a -q) ; docker rm $(docker ps -a -q) -f ; docker rmi $(docker images -q) -f",
"docker-tag": "docker tag $npm_package_config_servername $npm_package_config_dockerhubusername/$npm_package_config_servername:latest",
```

**Heroku - Deploy Container via API**
<https://devcenter.heroku.com/articles/container-registry-and-runtime#api>
```
curl -n -X PATCH https://api.heroku.com/apps/$APP_ID_OR_NAME/formation \
  -d '{
  "updates": [
    {
      "type": "web",
      "docker_image": "$WEB_DOCKER_IMAGE_ID"
    },
    {
      "type": "worker",
      "docker_image": "$WORKER_DOCKER_IMAGE_ID"
    },
  ]
}' \
  -H "Content-Type: application/json" \
  -H "Accept: application/vnd.heroku+json; version=3.docker-releases"
```

**AWS Stack is too complicated for this app. Switching to heroku but keeping AWS Route53 service**
- References:
<https://github.com/netbears/traefik-cluster-ecs>
<https://medium.com/@peatiscoding/docker-compose-ecs-91b033c8fdb6>

0. Clone traefik-cluster-ecs Repository
<https://github.com/ncmd/traefik-cluster-ecs>

1. Create ECS Cluster
<https://us-west-1.console.aws.amazon.com/ecs/home?region=us-west-1#/clusters/create/new>
- EC2 Linux + Networking
- Create Empty Cluster
- Cluster Name: stb
- Create

2. Build Traefik Container
<https://github.com/ncmd/traefik-cluster-ecs#building-the-traefik-container>
- Edit "traefik_ecs.toml" file
```
clusters = ["stb"]
domain = "ecs.securethebox.us"
region = "us-west-1"
```
- Set Local ENV variables
```
export AWS_REGION='us-west-1'
export AWS_ACCOUNT_ID='your_aws_account_id'
export ECR_REPO_NAME='traefik'
```
- Install AWS CLI
```
curl "https://s3.amazonaws.com/aws-cli/awscli-bundle.zip" -o "awscli-bundle.zip"
unzip awscli-bundle.zip
sudo /usr/local/bin/python3.6 awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
```
- Generate New Secret Access Key
```
https://console.aws.amazon.com/iam/home?#/security_credentials
Access Keys > Create New Access Key
```
- Configure Local Configuration
```
aws configure
```
- Create Repository and Login
Start Docker Service Locally...
```
aws ecr create-repository --repository-name traefik
aws ecr get-login --no-include-email --region us-west-1
docker build -t traefik:latest .
docker tag traefik:latest your_aws_account_id.dkr.ecr.us-west-1.amazonaws.com/traefik:latest
docker push your_aws_account_id.dkr.ecr.us-west-1.amazonaws.com/traefik:latest
```
- Create Repoistory
```
(aws ecr create-repository --repository-name $ECR_REPO_NAME) || true
```

**start cluster**
```
ecs-cli up
```
**shutdown cluster**
```
ecs-cli down
```
**generate ecs supported docker-compose file**
```
python3.7 convert-docker-compose.py
```
**startup ecs services**
```
ecs-cli compose --file <generated docker-compose file> service up
```
**check running ecs processes**
```
ecs-cli ps
```