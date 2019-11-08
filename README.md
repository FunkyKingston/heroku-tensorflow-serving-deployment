# heroku-tensorflow-serving-deployment

This repo demonstrates *TensorFlow Serving with Docker*, including deploying the given Docker container to Heroku. Deploying to Heroku requires some slight changes from the standard TensorFlow Serving getting started example (at https://www.tensorflow.org/tfx/serving/docker). Specifically, we need to use the *$PORT* that Heroku provides (as per described at https://devcenter.heroku.com/articles/runtime-principles#web-servers). 

Inside the TensorFlow Serving Docker image, the *tensorflow_model_server* will be started and it will listen to requests. These are *predict requests*, sent to a particular machine learning model that is hosted by the model server. The job of the model is to output a prediction, based on the (json-formatted) input data that it receives. 


## A look inside the Dockerfile

**Please note:** The *Dockerfile* used in this repo is an adapted version of the official TensorFlow Serving Dockerfile from https://github.com/tensorflow/serving/blob/master/tensorflow_serving/tools/docker/Dockerfile. 

The Dockerfile ends with executing `CMD ["/usr/bin/tf_serving_entrypoint.sh"]`. Previously, the following has been inserted into the *tf_serving_entrypoint.sh* script:

`tensorflow_model_server --port=8500 --rest_api_port=$PORT \
--model_name=${MODEL_NAME} --model_base_path=${MODEL_BASE_PATH}/${MODEL_NAME}`

Thus, the entrypoint script starts the tensorflow_model_server and tells it to receive requests using its REST API on the *$PORT* designated by Heroku (in other words, we cannot use the default 8501). 

In the Dockerfile we have could specify environment variables `MODEL_BASE_PATH` and `MODEL_NAME`. They are used as input when starting tensorflow_model_server, and will affect the URL to which we send requests, and can be used to clarify what model we are communicating with. For this example, however, they are just set as their defaults, `/models` and `model`, respectively. 


### Choosing a TensorFlow model and copying it into the image

We could use this approach to serve any TensorFlow/Keras model, after saving it in a TensorFlow *SavedModel* format - https://www.tensorflow.org/guide/saved_model.

Here, a simple example model is used, taken from the TensorFlow Serving repo, https://github.com/tensorflow/serving, in the *tensorflow_serving\servables\tensorflow\testdata* subfolder. The *saved_model_half_plus_two_cpu/00000123* folder in this repo, which contains a *TensorFlow SavedModel*, was copied from there.

In our Dockerfile, this model is copied into the image by `COPY saved_model_half_plus_two_cpu /${MODEL_BASE_PATH}/${MODEL_NAME}`.


## Deploy to Heroku

`heroku container:login`

`heroku create funky-tensorflow-serving`

`heroku container:push web --app funky-tensorflow-serving`

`heroku container:release web --app funky-tensorflow-serving`

https://funky-tensorflow-serving.herokuapp.com/v1/models/model


## Query the deployed TensorFlow model using the predict API

To send a predict request to the deployed model, in a terminal, run:
`curl -d '{"instances": [1.0, 2.0, 5.0]}' -X POST http://funkykingston.herokuapp.com/v1/models/model:predict.`
