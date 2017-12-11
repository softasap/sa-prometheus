# sa-prometheus


[![Build Status](https://travis-ci.org/softasap/sa-prometheus.svg?branch=master)](https://travis-ci.org/softasap/sa-prometheus)


#Usage:

Simple:

```YAML

- {
    role: "sa-prometheus"
  }


```

Advanced:

```YAML

vars:

  some_custom_properties:
    - {regexp: "^;allow_sign_up =(.*)", line: "allow_sign_up=false"}


roles:
  - {
      role: "sa-prometheus",

      option_install_go: true,
      go_version: 1.6,


      prometheus_user:   prometheus,
      prometheus_group:  prometheus,

      prometheus_base_dir: /opt/prometheus,
      prometheus_data_dir: "{{prometheus_base_dir}}/data",

      prometheus_version:                 2.0.0,
      prometheus_node_exporter_version:   0.15.2,
      prometheus_alertmanager_version:    0.11.0,

      prometheus_config_path:          /etc/prometheus
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


usage with ansible-galaxy workflow
----------------------------------

If you installed the `sa-prometheus` role using the command


`
   ansible-galaxy install softasap.sa-prometheus
`

the role will be available in the folder `library/softasap.sa-prometheus`
Please adjust the path accordingly.

```YAML

     - {
         role: "softasap.sa-prometheus"
       }

```


Copyright and license
---------------------

Code is dual licensed under the [BSD 3 clause] (https://opensource.org/licenses/BSD-3-Clause) and the [MIT License] (http://opensource.org/licenses/MIT). Choose the one that suits you best.

Reach us:

Subscribe for roles updates at [FB] (https://www.facebook.com/SoftAsap/)

Join gitter discussion channel at [Gitter](https://gitter.im/softasap)

Discover other roles at  http://www.softasap.com/roles/registry_generated.html

visit our blog at http://www.softasap.com/blog/archive.html
