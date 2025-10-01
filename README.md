# train_yolo11x
documentation on training yolo11x detector model with UAV images

on icefish

directory train-drone-model

setup

    pyenv shell 3.11.6
    python3 -m venv train-drone
    source train-drone/bin/activate
    pip install mbari-aidata

use yaml file from AI documentation

set TATOR_TOKEN

    export TATOR_TOKEN=<blahblahblah>  # token value from TATOR credentials
    echo $TATOR_TOKEN
    
script to download data

        aidata download dataset \
          --config https://docs.mbari.org/internal/ai/projects/config/config_uav.yml \
          --base-path $PWD/Sept232025 --voc \
          --labels "Batray","Bird","Boat","Cement_Ship","Egregia","Fish","Jelly","Kayak","Kelp","Mola","Mooring_Buoy","Otter","Person","Pinniped","Secci_Disc","Shark","Surfboard","Velella_velella","Velella_velella_raft","Whale" \
          --single-class "object" \
          --verified \
          --token $TATOR_TOKEN \
          --disable-ssl-verify

logs go to

        ~/mbari_aidata/logs/

./transform.sh

        aidata transform voc --base-path $PWD/Sept232025 --resize 640 --crop-size 640  --crop-overlap 0.5 --min-visibility 0.0 --min-dim 20


./voc_to_yolo.sh

        aidata transform voc-to-yolo  --base-path $PWD/Sept232025/transformed

./split.sh

first, activate conda environment for deepsea-ai

        conda activate deepsea-ai

then run deepsea-ai split

        deepsea-ai split -i $PWD/Sept232025/transformed -o $PWD/Sept232025split

# Train yolo11x model



upload or create data.yaml file

        train: /content/datasets/train/images
        val: /content/datasets/val/images
        test: /content/datasets/test/images
        
        nc: 1
        names: ['object']
        
        roboflow:
          workspace: liangdianzhong
          project: -qvdww
          version: 3
          license: CC BY 4.0
          url: https://universe.roboflow.com/liangdianzhong/-qvdww/dataset/3

# Train model from COCO weights  https://docs.ultralytics.com/models/yolo11/ 

In Colab notebook, upload data to google drive. Here it is named "uavs"

<img width="1450" height="372" alt="Screenshot 2025-10-01 at 2 54 41 PM" src="https://github.com/user-attachments/assets/b93e7812-a574-4997-a289-05553f39c937" />


and then move data to appropriate directories
        
        !mkdir {HOME}/datasets
        %cd {HOME}/datasets
        from google.colab import userdata
        uavs_folder = "/content/drive/MyDrive/uavs/"

        !mkdir /content/datasets/
        !mkdir /content/datasets/savedir/
        !cp -r "/content/drive/MyDrive/uavs/images.tar.gz" "/content/datasets/savedir/"

        !cp -r "/content/drive/MyDrive/uavs/labels.tar.gz" "/content/datasets/savedir/"

        !tar xf /content/datasets/savedir/images.tar.gz --directory /content/datasets/savedir/

        !tar xf /content/datasets/savedir/labels.tar.gz --directory /content/datasets/savedir/

move the data

        ## make the directories that yolo11 expects
        !mkdir /content/datasets/train/
        !mkdir /content/datasets/train/images/
        !mkdir /content/datasets/train/labels/
        !mkdir /content/datasets/test/
        !mkdir /content/datasets/test/images/
        !mkdir /content/datasets/test/labels/
        !mkdir /content/datasets/val/
        !mkdir /content/datasets/val/images/
        !mkdir /content/datasets/val/labels/
        
        #get the data.yaml file
        !cp "/content/drive/MyDrive/uavs/data.yaml" "/content/datasets/data.yaml"
        !ls /content/datasets/
        
        #move the data to the expected directories
        !cp -r "/content/datasets/savedir/images/train/" "/content/datasets/train/images/"
        !cp -r "/content/datasets/savedir/labels/train/" "/content/datasets/train/labels/"
        
        !cp -r "/content/datasets/savedir/images/test/" "/content/datasets/test/images/"
        !cp -r "/content/datasets/savedir/labels/test/" "/content/datasets/test/labels/"
        
        !cp -r "/content/datasets/savedir/images/val/" "/content/datasets/val/images/"
        !cp -r "/content/datasets/savedir/labels/val/" "/content/datasets/val/labels/"
        
        !ls /content/datasets/

Run the training 

        !yolo task=detect mode=train model=yolo11x.pt data=/content/datasets/data.yaml epochs=40 patience=5 imgsz=640 plots=True

Here are examles of yolo11x training commands, with different starting weights

        # Build a new model from YAML and start training from scratch
        yolo detect train data=coco8.yaml model=yolo11x.yaml epochs=100 imgsz=640
        
        # Start training from a pretrained *.pt model
        yolo detect train data=coco8.yaml model=yolo11x.pt epochs=100 imgsz=640
        
        # Build a new model from YAML, transfer pretrained weights to it and start training
        yolo detect train data=coco8.yaml model=yolo11x.yaml pretrained=yolo11x.pt epochs=100 imgsz=640

Save the results of training

        !cp "/content/runs/detect/train/weights/best.pt" "/content/drive/MyDrive/uavs/best.pt"
        !cp "/content/runs/detect/train/weights/last.pt" "/content/drive/MyDrive/uavs/last.pt"
        !cp -r "/content/runs/detect/train/" "/content/drive/MyDrive/uavs/train/"

NOTE: The results of the completed training are saved in {HOME}/runs/detect/train/. Let's examine them.

        !ls {HOME}/runs/detect/train/
        from IPython.display import Image as IPyImage
        IPyImage(filename=f'{HOME}/runs/detect/train/confusion_matrix.png', width=600)
        IPyImage(filename=f'{HOME}/runs/detect/train/results.png', width=600)
        IPyImage(filename=f'{HOME}/runs/detect/train/val_batch0_pred.jpg', width=600)

Validate the trained model

        !yolo task=detect mode=val model=/content/runs/detect/train/weights/best.pt data=/content/datasets/data.yaml

Inference the test set with the trained model

        !yolo task=detect mode=predict model=/content/runs/detect/train/weights/best.pt conf=0.25 source=/content/datasets/test/images/test/ save=True
