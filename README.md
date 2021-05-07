Yolov5_torchserve
====

This folder contains scripts to setup [Torchserve](https://github.com/pytorch/serve) for [Yolov5](https://github.com/ultralytics/yolov5), popular open-source object detection method.

Usage
===========

       1. Create env

           sh create_env.sh
           Note: assume os is linux with cuda 11.0 

       2. Download the model from YoloV5 releases
          
          cd yolov5_mar
          sh download_model.sh

       3. Archive the model files

           cd yolov5_mar
           zip -r ../yolo5.mar .
           cd ..

       4. Host the yolov5 model

           mkdir model_store
           mv yolo5.mar model_store/.

       5. Run the server
          
          nohup sh start_server.sh &

       6. Check the healthy
         
          curl "http://localhost:8081/models/yolo5"

          or check logs/ts_log.log

       7. Test object detection API

          curl -X POST http://127.0.0.1:8080/predictions/yolo5 -T kitten.jpg

Inference with gRPC
==========

      1. Install dependencies (see no. 1, 2 in https://github.com/pytorch/serve#install-torchserve-and-torch-model-archiver)

      2. Install gRPC Python dependencies
         pip install -U grpcio protobuf grpcio-tools

      3. Generate inference client using proto files
         python -m grpc_tools.protoc --proto_path=./proto/ --python_out=./proto/ --grpc_python_out=./proto/ ./proto/inference.proto ./proto/management.proto
         
      4. Run inference using a sample client 
         python proto/torchserve_grpc_client.py infer yolo5 kitten.jpg

Performance
===========

[Coco 2017 Val Images(5K)](https://cocodataset.org/#download) is used to test throughput/latency in the above setup.

       1. Server machine: Intel xeon Gold 6159 CPU@2.1, 2 Tesla P4 with CentOS 7
       2. Test setup
          a. 10 http clients in parallel. Each client which runs in other machine sends 500 images sequentially. 
             Throughput: 26 images/second  Average latency: 384 millisec/request

          b. Native torch program (yolov5_mar/simple_batch_test.py) without torchserve. Run two processes (batch_size: 10). Each one has own gpu and dataset (#images: 2500). 
             Throughput: 26 images/second.
          
Reference
===========

1. [Torchserve](https://pytorch.org/serve/)
2. [Ultralytics/yolov5](https://github.com/ultralytics/yolov5)
