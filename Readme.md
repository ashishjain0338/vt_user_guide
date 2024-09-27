# VT-SRIB Joint Project User-Guide & TroubleShooting:
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
2. xApp-Management:
    1. Deployment
    ```bash
        cd ~/work_dir/qos-xapp/xapp-descriptor
        ./run.sh
    ```
    2. UnDeployment
    ```bash
        # Uninstalling xApp
        dms_cli uninstall --xapp_chart_name=qos-xapp --namespace=ricxapp
        # Deregister the xApp from NearRtRic
        export APP_SVC=$(kubectl get service -n ricplt | grep appmgr-http| awk '{print $3}')
        curl -v -X POST "http://$APP_SVC:8080/ric/v1/deregister" -H "accept: application/json" -H "Content-Type: application/json" -d '{"appName": "qos-xapp","appInstanceName":"qos-xapp"}' 
    ```
    
    3. List All registered xApps
       ```bash
       export APP_SVC=$(kubectl get service -n ricplt | grep appmgr-http| awk '{print $3}')
       curl http://$APP_SVC:8080/ric/v1/xapps| jq .
       ```

3. Starting xApp (Manually):
    ```bash
        xAPP_POD=$(kubectl get pods -n ricxapp -l app=ricxapp-qos-xapp -o jsonpath='{.items[*].metadata.name}')
        kubectl exec -it $xAPP_POD -n ricxapp bash
        python3 src/xapp.py
    ```
4. TroubleShooting & Solution:
    1. A1-Policy is sent but not recieved by xAPP:
    ```bash
        # Delete the old xApp or rerun the current one
        xAPP_POD=$(kubectl get pods -n ricxapp -l app=ricxapp-qos-xapp -o jsonpath='{.items[*].metadata.name}')
        kubectl delete pods $xAPP_POD -n ricxapp
    ```
    2. RIC Control Message Sent Status is  `False`
        The message suggests that the message is not sent from xApp, signifying that there is some issue with RMR messaging and route table.
        ```bash
        # Check the Routing table and meid to debug more
        export RTM_SVC=$(kubectl get service -n ricplt | grep rtmgr-http| awk '{print $3}')
        curl http://$RTM_SVC:3800/ric/v1/getdebuginfo| jq .
        ```
    3. RIC Control Message Sent Status is  `True` but message is not recieved over gnb
        ```bash
        # Check the E2TERM-logs and if it says Notification-handler not found, Then wait for 30-60 seconds (It will come-up)

        E2TERM_POD=$(kubectl get pods -n ricplt -l app=ricplt-e2term-alpha -o jsonpath='{.items[*].metadata.name}')
        kubectl logs $E2TERM_POD -n ricplt --tail 100 -f
        ```
    4. Message was sent to Gnb but gnb crashes saying `RAN function ID not supported 148`
        
        The above message suggests "RAN function" was not registered signifying that NearRtRic and Gnb wasn't successfully connected
        Solution: Restart the GNB
        ```bash
            During Connection to NearRtRic (The logs of Gnb Must say:)
            ADDING RAN FUNCTION 147 | oid --> 1.3.6.1.4.1.53148.1.2.2.2
            ADDING RAN FUNCTION 148 | oid --> 1.3.6.1.4.1.53148.1.1.2.3
        ```
    5. If Gnb is not able to register
        ```bash
            # Delete the E2-term Pod
            E2TERM_POD=$(kubectl get pods -n ricplt -l app=ricplt-e2term-alpha -o jsonpath='{.items[*].metadata.name}')
            kubectl delete -f $E2TERM_POD -n ricplt
        ```
Note: Generally, It takes about 30-45 seconds for the messaging to start from xApp to Gnb after starting xApp (manually)
