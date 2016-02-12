.. image:: https://api.travis-ci.org/shinken-monitoring/mod-ws-arbiter.svg?branch=master
  :target: https://travis-ci.org/shinken-monitoring/mod-ws-arbiter
.. _ws_daemon_module:

===================
Web Service Module 
===================


The ws-arbiter module is an Arbiter (or Receiver) module to treat requests from remote hosts via HTTP(s). 


Configuring the Web Service Module 
===================================

The module configuration is made in the ws-arbiter.cfg file, in the modules configuration directory. To enable this module, simply add the module to your Arbiter or Receiver daemon configuration.

If data is POSTed to this page, it will validate the data to determine if it is a service or host check result and submit the data to the Shinken external command queue.

This module listens by default on all IP interfaces, TCP port 7760 and it supports anonymous or authenticated access. This configuration may be changed in the ws-arbiter.cfg file.


Using the Web Service Module 
=============================


The web service listens for POSTs to:

 - /restart
  Makes Shinken restart all daemons (/etc/init.d/shinken restart)

  Command:
  ::
    curl -u user:password -d '' http://shinken-srv:7760/restart


 - /reload
  Makes Shinken reload configuration (/etc/init.d/shinken reload)
 
  Command:
  ::
    curl -u user:password -d '' http://shinken-srv:7760/reload


 - /acknowledge
  Sets/removes an acknowledge for an host/service

  Parameters:
   - action: (default = add)
    add, to add an acknowledge for an host/service
    delete, to remove current acknowledges on host/service
    
   - host_name:
    Host name
    
   - service_description: (default = '' for host acknowledge only)
    Service description
    
   - time_stamp: (default = current time)
    
   - sticky: (default = 1)

   - notify: (default = 0)

   - persistent: (default = 1)

   - author: (default = 'anonymous')

   - comment: (default = 'No comment')

 
  Command:
  ::
    curl -u user:password -d "&host_name=host-ack&service_description=service-ack&author=Me&comment=Ack problem" http://shinken-srv:7760/acknowledge

 - /downtime
  Sets/removes a downtime for an host/service


  Parameters:
   - action: (default = add)
    add, to add an acknowledge for an host/service
    delete, to remove current downtimes on host/service
    
   - host_name:
    Host name
    
   - service_description: (default = '' for host acknowledge only)
    Service description
    
   - time_stamp: (default = current time)
    
   - start_time: (default = current time)
    
   - end_time: (default = current time)
    
   - fixed: (default = 1)

   - duration: (default = 86400 seconds)

   - trigger_id: (default = 0)

   - author: (default = 'anonymous')

   - comment: (default = 'No comment')

 
  Command:
  ::
    curl -u user:password -d "&host_name=host-ack&author=Me&comment=Downtime host" http://shinken-srv:7760/downtime

 - /push_check_result
  Sends checks result to the Arbiter. Use curl or embed the HTTP calls in your software to submit check results.

  Command:
  ::
    curl -u user:password -d "time_stamp=$(date +%s)&host_name=host-checked&service_description=service-checked&return_code=0" --data-urlencode "output=Everything OK" http://shinken-srv:7760/push_check_result

  Example with more readability:
    
  ::
    curl 
    -u user:password 
    -d "time_stamp=$(date +%s)
    &host_name=host-checked
    &service_description=service-checked
    &return_code=0"
    --data-urlencode "output=Everything OK"
    http://shinken-srv:7760/push_check_result

 - /push_checks_perfdata
 Sends checks *Performance Data* to the Arbiter.
 This is actually almost the same than /push_check_result but limited to only submitting performance
  data without any return_code, nor output, associated with it.

 The checks perfdata have to be sent as a JSON array of dicts.
 Each dict represents a single check perfdata for a particular host or service.

 The dicts have to have the following keys/values :
  - hostname : The usual hostname.  REQUIRED.
  - service_description : The usual service_description. OPTIONAL.
    If unset, or empty/falsy, then we consider this is an host check perfdata.
    Otherwise it's a service check perfdata.
  - time : The UNIX time when the check was done. OPTIONAL, if not set then WS-Arbiter will
    default to time.time().
  - perfdata : The actual "perfdata" of the check.  REQUIRED.
    See:
      http://shinken.readthedocs.org/en/latest/07_advanced/perfdata.html
      https://www.monitoring-plugins.org/doc/guidelines.html#AEN201
      http://docs.icinga.org/latest/en/perfdata.html#formatperfdata
    this is so a list of metric results *space* separated.

 Command:
 ::
    curl -X POST -H "Content-Type: application/json"  \
    --user username:password  \
    -d 'THE_ACTUAL_JSON_DATA' \
    http://shinken-srv:7760/push_checks_perfdata


    Examples:

    # send one perfdata, related to the service "ping" on the host "host1",
    #   The perfdata contains 2 metrics:
    #       percent_packet_loss=0
    #   and
    #       rta=0.80

    $ curl -X POST -H "Content-Type: application/json"  \
      --user username:password \
      -d '[{"hostname":"host1","service_description":"ping","perfdata":"percent_packet_loss=0 rta=0.80"}]' \
      http://shinken-srv:7760/push_checks_perfdata


    # send two perfdata,
    #   The first is for the 'http' service check on the host "host1",
    #     the perfdata for it contains 3 metrics.
    #   The second perfdata is for an host check, for the host "host2",
    #     the perfdata for it contains 2 metrics.

    $ curl -X POST -H "Content-Type: application/json"  \
    --user username:password  \
    -d '[\
    {"hostname":"host1","service_description":"http","perfdata":"rsp_time=0.1s max_speed=60 avg_speed=50"},\
    {"hostname":"host2","perfdata":"percent_packet_loss=0.3 rta=0.80"}\
    ]'  \
    http://shinken-srv:7760/push_checks_perfdata
