## Consuming LitmusChaos Prometheus Metrics in Dynatrace

This document consists of the basic steps involved in setting up the chaos prometheus metrics to be consumed in Dynatrace, along with some suggestions on 
how these chaos metrics can be used alongside native dynatrace metrics (infra or application) to identify the impact of the fault injection. 

This doc assumes that Dynatrace is already setup with OneAgent agents/daemonset running on the cluster nodes (Ref: [Dynatrace Setup for Kubernetes](https://www.dynatrace.com/support/help/setup-and-configuration/setup-on-container-platforms/kubernetes/set-up-k8s-monitoring))

## PreRequisites

- Ensure that you are using/running an ActiveGate that supports scraping of prometheus metrics by the OneAgent (ActiveGate 1.217+) 
- Ensure that you have **Monitoring Enabled** for the Kubernetes cluster added as a tenant in Dynatrace
- Enable the setting **Turn on Monitor annotated Prometheus exporters** for your Kubernetes cluster

### References: 

![image](https://user-images.githubusercontent.com/21166217/149325098-86f41515-d0c4-4456-90f0-6497aa85d362.png)
![image](https://user-images.githubusercontent.com/21166217/149324845-ecf07cf7-7ad6-41c6-a77f-606d7739d6a9.png)

## Annotate the Chaos Exporter pod to enable mterics scrape by Dynatrace 

- This section assumes you already have the Litmus control plane (chaoscenter) and execution plane (subscriber et al) running in your Kubernetes cluster. (Ref: [LitmusChaos Setup](https://docs.litmuschaos.io/docs/getting-started/installation))
- Add these annotations to the chaos-exporter pod: 

  ```
  metrics.dynatrace.com/scrape: 'true' 
  metrics.dynatrace.com/port: 8080
  ```

  Ex: `kubectl annotate pod <chaos-exporter-pod>` metrics.dynatrace.com/scrape=true -n <litmus-agent-namespace>`, `kubectl annotate pod <chaos-exporter-pod> metrics.dynatrace.com/port=8080 -n <litmus-agent-namespace>`
  
  **Note** You can also add the annotations to the corresponding service, provided it is in the same namespace as the exporter pod. 
  
  For more info on annotations based metrics scrape, ref: [Monitoring Prometheus Metrics From Kubernetes in Dynatrace](https://www.dynatrace.com/support/help/how-to-use-dynatrace/infrastructure-monitoring/container-platform-monitoring/kubernetes-monitoring/monitor-prometheus-metrics#best)
  
  ## Viewing Chaos Metrics in Dynatrace
  
  - While Litmus provides several [metrics](https://github.com/litmuschaos/chaos-exporter), probably the most useful one is `litmus_awaited_experiments`. This metric can be used 
  (along with appropriate filters/labels), to indicate the period when the experiment is active. Hence, it can be used in conjunction with other infra/app metrics
  in a common dashboard to view exactly how the infra/app behaves during the fault injection period. 
  
  - The chaos metrics can be viewed using the **Data Explorer** window/tab on Dynatrace, where you can add important infra/app metrics along with chaos metrics to create (& save)
  custom dashboards. 
  
  ![image](https://user-images.githubusercontent.com/21166217/149329653-0b0d3d27-2012-4c5a-9cc2-8062f6a5a32a.png)
  
  - In the simple example below shows a custom dashboard that tracks CPU utilization on an app container during a cpu-hog chaos experiment: 
  
  ![image](https://user-images.githubusercontent.com/21166217/149329490-9c243e1c-dae1-43a1-806e-c23f14e94023.png)
  
## Other Information

- Prometheus chaos metrics can also be used to configure alerts in Dynatrace. The `litmuschaos_experiment_verdict` with appropriate labels (ex: chaosresult_verdict="Pass" or "Fail") 
that can be used to notify the SREs/Dev teams of the result of an experiment. 

- Litmus also creates ganerates Kubernetes events for the ChaosEngine & ChaosResult objects. These can be ingested by enabling "Monitor Events" settings: 

![image](https://user-images.githubusercontent.com/21166217/149330309-8d7e7051-bbdb-4502-aa56-acbb7fa53716.png)

  For more info on ingesting events, ref: [Monitoring Kubernetes Events in Dynatrace](https://www.dynatrace.com/support/help/how-to-use-dynatrace/infrastructure-monitoring/container-platform-monitoring/kubernetes-monitoring/monitor-events-kubernetes)
  
  

