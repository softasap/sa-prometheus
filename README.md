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

