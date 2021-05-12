## Google Cloud Storage setup

1. Go to https://console.cloud.google.com/
1. Press on side bar > Go to Cloud Storage > Create a bucket with Uniform Permission for easy handling
1. Press on side bar > Go to APIs & Services > Credentials 
1. Press on `Create Credentials` and make a Service account with permissions `Storage Legacy Bucket Owner` and `Storage Legacy Object Owner`. (You may need to assign it within the Cloud Storage page)
1. Download a `Service Account` key.

1. _INSIDE THE CODE_ Change the default bucket or pass in the bucket name when calling the `GStorage` class
1. _INSIDE THE CODE_ Update `conf.php` to point to the `Service Account` key downloaded earlier

## Credits: 

Github: https://github.com/NanoCode012/DistributedSetup