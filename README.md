# train_yolov11x
documentation on training yolov11x detector model with UAV images

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
