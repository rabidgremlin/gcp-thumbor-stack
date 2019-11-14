# GCP Thumbor Stack

Deployment Manager scripts to create an end to end [Thumbor](http://thumbor.org/) image resizing and cropping stack on the Google Cloud Platform

This script creates a HTTP Cloud Load Balancer, pointing at a cluster of Thumbor services distributed across one or more zones. 
The Thumbor servers are configured to process images from a storage bucket and the whole stack is front-ended by a Cloud CDN for speed and scalability. 

## Create and test the stack
In your Cloud Shell, run the following commands. You can run replace ```thumbor-test``` with your own name. All resources created by the deployment manager scripts are pre-fixed with this name.

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

You can also simulate load on the stack by:
```
sudo apt-get -y install apache2-utils
ab -n 100 -c 2 http://[IP_ADDRESS]/unsafe/500x200/smart/smiling-man-and-woman-sitting-on-pavement-2216727.jpg
```
If you change the parameters in the URL and rerun multiple times you can see the speed improvements 
as the resized image is cached by Cloud CDN.

You can also force your cluster to scale by deliberate forceing cache misses:
```
ab -n 100 -c 5 http://[IP_ADDRESS]/unsafe/`date +%N | cut -c 7-`x200/smart/smiling-man-and-woman-sitting-on-pavement-2216727.jpg
```

## Deleting the stack
First remove any images from your storage bucket, then run:
```
gcloud deployment-manager deployments delete thumbor-test
```

## Make production ready

For production:

* Adjust the ```zones``` and ```storageLocation``` values as needed
* You need to set the ```securityKey``` to a unique value
* You will also most likely want to set ```allowUnsafe``` to ```false``` to require [signed URLs](https://thumbor.readthedocs.io/en/latest/security.html)
* You will configure DNS to point a domain at the IP address of the load balancer
* If you want to set up SSL you will need [configure a HTTPS front-end](https://cloud.google.com/load-balancing/docs/https/) and [a SSL certificate](https://cloud.google.com/load-balancing/docs/ssl-certificates)
