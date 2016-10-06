# sa-prometheus


[![Build Status](https://travis-ci.org/softasap/sa-prometheus.svg?branch=master)](https://travis-ci.org/softasap/sa-prometheus)


#Usage:

Simple:

```

- {
    role: "sa-prometheus"
  }


```

Advanced:

```

vars:

  some_custom_properties:
    - {regexp: "^;allow_sign_up =(.*)", line: "allow_sign_up=false"}


roles:
  - {
      role: "sa-prometheus"
    }


```


Best cookied with Grafana (example included in box example , see related article on  https://www.digitalocean.com/community/tutorials/how-to-add-a-prometheus-dashboard-to-grafana)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_011.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_012.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_013.png)

![alt tag](https://raw.githubusercontent.com/softasap/sa-prometheus/master/box-example/docs/Selection_014.png)


Further reading:


https://www.digitalocean.com/community/tutorials/how-to-query-prometheus-on-ubuntu-14-04-part-1

https://www.digitalocean.com/community/tutorials/how-to-query-prometheus-on-ubuntu-14-04-part-2


