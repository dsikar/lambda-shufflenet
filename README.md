# AWS Lambda function for Shufflenet

## Setting up

This follows the procedure given in 
```
https://github.com/dsikar/lambda-tensorflow-example
```
in turn following the procedure given in
```
https://aws.amazon.com/blogs/machine-learning/using-container-images-to-run-tensorflow-models-in-aws-lambda/  
```
## Shufflenet

The Shufflenet network is implemented in
```
https://github.com/NeelBhowmik/efficient-compact-fire-detection-cnn
```
## Adapting Shufflenet to run as AWS Lambda function

The script [inference_ff_lambda.py](https://github.com/dsikar/lambda-shufflenet/blob/master/inference_ff_lambda.py) in modified from [inference_ff.py](https://github.com/NeelBhowmik/efficient-compact-fire-detection-cnn/blob/main/inference_ff.py), where the impor
```
from models import shufflenetv2
```
is omitted, and the code incorporated into [inference_ff_lambda.py](https://github.com/dsikar/lambda-shufflenet/blob/master/inference_ff_lambda.py), such that import is not required. This is due to an AWS Lambda environment constraint of not being able to import local modules into the python environment, even though the model can be copied (from S3 using *boto*) to the local filesystem. This is assumed to be because the import fires before the module is copied to disk from S3:
```
[ERROR] FileNotFoundError: [Errno 2] No such file or directory: '/models/shufflenetv2.py.CecCA8aB'
Traceback (most recent call last):
  File "/var/lang/lib/python3.8/imp.py", line 234, in load_module
    return load_source(name, filename, file)
  File "/var/lang/lib/python3.8/imp.py", line 171, in load_source
    module = _load(spec)
  File "<frozen importlib._bootstrap>", line 702, in _load
  File "<frozen importlib._bootstrap>", line 671, in _load_unlocked
  File "<frozen importlib._bootstrap_external>", line 848, in exec_module
  File "<frozen importlib._bootstrap>", line 219, in _call_with_frames_removed
  File "/var/task/inference_ff_lambda.py", line 46, in <module>
    result = client_s3.download_file("dsikar.models.bucket",'shufflenetv2.py', "/models/shufflenetv2.py")
  File "/var/runtime/boto3/s3/inject.py", line 170, in download_file
    return transfer.download_file(
  File "/var/runtime/boto3/s3/transfer.py", line 307, in download_file
    future.result()
  File "/var/runtime/s3transfer/futures.py", line 106, in result
    return self._coordinator.result()
  File "/var/runtime/s3transfer/futures.py", line 265, in result
    raise self._exception
  File "/var/runtime/s3transfer/tasks.py", line 126, in __call__
    return self._execute_main(kwargs)
  File "/var/runtime/s3transfer/tasks.py", line 150, in _execute_main
    return_value = self._main(**kwargs)
  File "/var/runtime/s3transfer/download.py", line 571, in _main
    fileobj.seek(offset)
  File "/var/runtime/s3transfer/utils.py", line 367, in seek
    self._open_if_needed()
  File "/var/runtime/s3transfer/utils.py", line 350, in _open_if_needed
    self._fileobj = self._open_function(self._filename, self._mode)
  File "/var/runtime/s3transfer/utils.py", line 261, in open
    return open(filename, mode)
```
The line
```
result = client_s3.download_file("dsikar.models.bucket",'shufflenetv2.py', "/models/shufflenetv2.py")
```
showing the file is being copied to a **/models** directory. It remains to be determined if copying to **/tmp** would work, where the temporary tmp folder is a feature of Lambda and S3, and no other directory names can be created, then in the python script importing from tmp
```
from tmp import shufflenetv2
```
as the original path used for weights had also to be modified such that S3 and boto and the Python script could agree on the location
```
result = client_s3.download_file("dsikar.models.bucket",'shufflenet_ff.pt', "/tmp/shufflenet_ff.pt")
```
Note, this failed with
```
result = client_s3.download_file("dsikar.models.bucket",'shufflenet_ff.pt', "/weights/shufflenet_ff.pt")
```

## Increasing memory available 
The prediction also errored while the available memory was set to default 128MB. This was doubled
```
aws lambda update-function-configuration --function-name lambda-shufflenet --memory-size 256
```

## Inconsistency of AWS Lambda/S3 compared to local environment

Lambda predications are inconsistent with local predictions.
Comments to follow.
