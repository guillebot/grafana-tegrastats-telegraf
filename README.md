# grafana-tegrastats-telegraf

I'm using the nvidia jetson xavier platform.

In order to remote monitor it I use [influxdb](https://www.influxdata.com). I send the metrics using Telegraf, and visualize them using Grafana.

For the most part I use [This beautiful](https://grafana.com/grafana/dashboards/8003) dashboard, but for the more specific parts like temperature monitoring, GPU, encoders, etc. I'm going with the nvidia tool [tegrastat](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3231/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/AppendixTegraStats.html)

## So the basic workflow is:

### Generate the logs:

`tegrastats tegrastats --interval 10000 --logfile /var/log/tegrastat`

### Read the logs with telegraf input.tail plugin:

Excerpt from `/etc/telegraf/telegraf.conf`
```
[[inputs.tail]]
  ## file(s) to tail:
  files = ["/var/log/tegrastat"]
  from_beginning = false
  data_format = "grok"

  #name of the "Metric" (which I want to see in Grafana eventually)
  name_override = "tegrastat"

 grok_patterns = ["%{CUSTOM_LOGS}"]

 grok_custom_patterns = '''
CUSTOM_LOGS %{NUMBER:ramused:int}/%{NUMBER:ramtotal:int}MB \(lfb %{NUMBER:pages:int}x4MB\) CPU \[%{NUMBER:cpu1percentaje:int}\%@%{NUMBER:cpu1freq:int},%{NUMBER:cpu2percentaje:int}\%@%{NUMBER:cpu2freq:int},%{NUMBER:cpu3percentaje:int}\%@%{NUMBER:cpu3freq:int},%{NUMBER:cpu4percentaje:int}\%@%{NUMBER:cpu4freq:int},%{NUMBER:cpu5percentaje:int}\%@%{NUMBER:cpu5freq:int},%{NUMBER:cpu6percentaje:int}\%@%{NUMBER:cpu6freq:int},%{NUMBER:cpu7percentaje:int}\%@%{NUMBER:cpu7freq:int},%{NUMBER:cpu8percentaje:int}\%@%{NUMBER:cpu8freq:int}\] EMC_FREQ %{NUMBER:emcfreqpercentaje:int}\%@%{NUMBER:emcfreq:int} GR3D_FREQ %{NUMBER:gr3dfreqpercentaje:int}\%@%{NUMBER:gr3dfreq:int} NVENC %{NUMBER:nvencfreq:int} NVENC1 %{NUMBER:nvenc1freq:int} APE %{NUMBER:ape:int} MTS fg %{NUMBER:mts_fg:int}\% bg %{NUMBER:mts_bg:int}\% AO@%{NUMBER:ao_temp:float}C GPU@%{NUMBER:gpu_temp:float}C Tboard@%{NUMBER:tboard_temp:float}C Tdiode@%{NUMBER:tdiode_temp:float}C AUX@%{NUMBER:aux_temp:float}C CPU@%{NUMBER:cpu_temp:float}C thermal@%{NUMBER:thermal_temp:float}C PMIC@100C GPU %{NUMBER:gpupowercur:int}/%{NUMBER:gpupoweravg:int} CPU %{NUMBER:cpupowercur:int}/%{NUMBER:cpupoweravg:int} SOC %{NUMBER:socpowercur:int}/%{NUMBER:socpoweravg:int} CV %{NUMBER:cvpowercur:int}/%{NUMBER:cvpoweravg:int} VDDRQ %{NUMBER:vddrqpowercur:int}/%{NUMBER:vddrqpoweravg:int} SYS5V %{NUMBER:sys5vpowercur:int}/%{NUMBER:sys5vpoweravg:int}
'''
```

### Then you can of course graph them, put alarms, etc as usual.


## Notes:

I didn't know [grok](https://code.google.com/archive/p/semicomplete/wikis/Grok.wiki), which is a nice line-parser a la regex.
Here a couple of tools that I used to assemble the custom pattern:
https://grokdebug.herokuapp.com/
http://grokconstructor.appspot.com/do/match#result


