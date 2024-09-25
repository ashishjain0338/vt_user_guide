# VT Project User-Guide & TroubleShooting:
## A) Core:
1. VM-Ip: 172.167.0.203
2. Deployment:
    ```bash
      ~/script/run_core.sh
    ```
3. Build:
    ```bash
      cd ~/ashish/ash_open5gs
      # rm -rf build/  # For Fresh Build
      ./build-open5gs.sh
    ```
    Note: Make sure after 'Fresh-build', the config file (~/ashish/ash_open5gs/build/configs/sample.yaml) contains right configuration

4. Logging:
    
    To check currently connected-ue's and their PDU-Ip's
    ```bash
      cat ~/ashish/ash_open5gs/build/tests/app/pdu_info.txt
    ```
5. TroubleShooting:
    1. Killing Core:
     ```bash
        ps -e|grep open5gs|awk '{print $1}'| xargs kill
     ```

   
## B) Gnb:
1. VM-Ip: 172.167.0.203
2. Deployment:
  ```bash
    ~/script/run_gnb.sh
  ```
3. Build:
  ```bash
    cd ~/ashish/ash_srsRAN_project/
    # rm -rf build/  # For Fresh Build
    ./build-ran.sh
  ```

4. Logging:

    Kpi logs will be found under
    ```bash
    cat ~/ashish/ash_srsRAN_project/build/apps/gnb/dlprb_slice.csv
    ```
5. TroubleShooting:
    1. Killing Gnb:
     ```bash
        ps -e|grep gnb|awk '{print $1}'| sudo xargs kill
     ```

## C) Kafka-Consumer & Pdu-Info-Writer:
1. VM-Ip: 172.167.0.203
2. Deployment(Kafka-Consumer):
  ```bash
    ~/script/run_kafka_consumer.sh
  ```
3. Deployment (Pdu-Info-Writer):
  ```bash
    # Starting the script to update Pdu-info in influxdb (to be shown in Grafana)
    ~/script/run_pdu_info_writer.sh
  ```
4. TroubleShooting:
    1. Creating Kafka Topic:
     ```bash
        ~/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <TOPIC>
     ```
    2. Deleting Kafka Topic:
    ```bash
        ~/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic <TOPIC>
     ```
    3. Reading Messages:
     ```bash
        ~/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic <TOPIC> --from-beginning
     ```
    4. Sending Messages Over Kafka Pipeline:
     ```bash
        echo "<Message Here>" | ~/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic <TOPIC>
     ```
     Note: In the setup, KPI are produced and consumed over `<TOPIC>` = test
 
4. Logging:

    Kafka logs will be found under
    ```bash
    cat ~/ashish/ash_srsRAN_project/build/apps/gnb/kafka-log.txt
    ```

## D) Ue-Traffic-Generation:
1. VM-Ip: 172.167.0.203
2. Traffic-Generation:
    1. Ue-Side-Client (TerMux) 
          ```bash
            iperf3 -s
          ```
    2. Gnb-Side-Traffic
        ```bash
            cd ~/script/traffic-gen/
             python3 gen-traffic.py <PDU-IP>
          ```
## E) Grafana:
1. VM-Ip: 172.167.1.215
2. SSH To Dashboard:
    ```bash
        ssh -L 31780:localhost:31780 cci@172.167.1.215
    ```
3. Working Around Dashboard:
    1. Url : http://localhost:31780/
    2. username: `admin` , pwd: `admin` 
    3. Dashboard-Url: http://localhost:31780/dashboards

Note: Since the Server-time runs 4hrs behind IST, therefore need to modify time-range as `{"from":"now-4h-5m","to":"now-4h"}`

## F) Non-RT-RIC 
1. VM-Ip: 172.167.1.215
2. Deploying rApp:
  ```bash
    # For Mode-1: (Train & Predict)
        kubectl apply -f ~/work_dir/rapp/cur_rapp/live_data/model_train_true/.
    # For Mode-2: (Use Existing & Predict)
        kubectl apply -f ~/work_dir/rapp/cur_rapp/live_data/model_train_false/.
  ```
  In order to delete, use `kubectl delete -f <KRM_folder>`
  
3. Starting rApp (Manually):
    ```bash
        kubectl exec -it <rAPP_POD> bash
        python3 qos-rapp/run_policy.py
    ```
 
4. Logging:

    Prediction Vs Actual can be seen in Grafana Dashboard (vt-testbed-debug)
    
## G) InfluxDb 
1. VM-Ip: 172.167.1.215
2. TroubleShooting:
    1. Running Queries:
     ```bash
        docker exec -it smo-influxdb bash
        influx
        use vt_slice
        <QUERY>
     ```
    2. Various Queries:
    ```bash
        # Get a list of all tables/measurements
            SHOW MEASUREMENTS
        # Creating a measurement is equivalent to inserting data
            INSERT <MEASUREMENT_NAME>,col1=val1 col2=val2
         # Reading data from measurement
            SELECT * FROM <MEASUREMENT_NAME>
         #  Dropping measurement (for-fresh-data-collection)
            DROP MEASUREMENT <MEASUREMENT_NAME>
     ```
   3. Dropping Measurement using a script from GNB-Vm (VM-Ip: 172.167.0.203)
        ```bash
            ~/script/delete_influx_records.sh <MEASUREMENT_NAME>
        ```
     Note: In the setup, KPI are stored over `<MEASUREMENT_NAME>` = live_data
     
## I) AIMLFW 
1. VM-Ip: 172.167.0.111
2. SSH To Dashboard:
    ```bash
        ssh -L 32005:localhost:32005 -L 32002:localhost:32002 -L 32088:localhost:32088 cci@172.167.0.111
    ```
3. Working Around Dashboard:
    1. AIML-Dashboard : http://localhost:32005/
    2. Pipelines-Jupyter-Notebook-Url: http://localhost:32088/tree?



## I) Near-RT-RIC 
1. VM-Ip: 172.167.3.3
2. Deploying xApp:
3. Starting xApp (Manually):
    ```bash
        xAPP_POD=$(kubectl get pods -n ricxapp -l app=ricxapp-qos-xapp -o jsonpath='{.items[*].metadata.name}')
        kubectl exec -it $xAPP_POD -n ricxapp bash
        python3 src/xapp.py
    ```
4. TroubleShooting:
        1. l
        2. d
        3. 

Note: Generally, It takes about 30-45 seconds for the messaging to start from xApp to Gnb after starting xApp (manually)

