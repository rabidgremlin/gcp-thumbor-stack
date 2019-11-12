# GCP Thumbor Stack

Deployment Manager scripts to create an end to end thumbor image resizing stack on the Google Cloud Platform

## Create the stack
Run the following command. You can run replace ```thumbor-test``` with your own name. All resources created by the deployment manager scripts are pre-fixed with this name.

In your Cloud Shell, run the following commands:

```
git https://github.com/rabidgremlin/gcp-thumbor-stack.git
cd gcp-thumbor-stack
gcloud deployment-manager deployments create thumbor-test --config deploy.yaml
```

Add an example image to your bucket. You will need to replace [BUCKET_NAME] with the name of storage bucket output by the deployment manager:
```
wget https://images.pexels.com/photos/2216727/pexels-photo-2216727.jpeg?fm=jpg -O smiling-man-and-woman-sitting-on-pavement-2216727.jpg
gsutil cp smiling-man-and-woman-sitting-on-pavement-2216727.jpg gs://[BUCKET_NAME]
```

Determine public IP address of CDN/Load balancer
```
gcloud compute forwarding-rules list
```

In a browser retrieve a resized image. Replace [IP_ADDRESS] with the IP address of your load balancer
```
http://[IP_ADDRESS]/unsafe/200x200/smart/smiling-man-and-woman-sitting-on-pavement-2216727.jpg
```
**Note:** You may have to wait several minutes after your stack has been created for this image retrieve to work
