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
