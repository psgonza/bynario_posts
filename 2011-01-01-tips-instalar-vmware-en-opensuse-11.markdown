Title: TIPS: Instalar VMWARE en OpenSuse 11
Date: 2011-01-01 00:48:00
Slug: 2011-01-01-tips-instalar-vmware-en-opensuse-11
Tags: linux,tips


Para instalar alguna de las diferentes versiones de VMWARE (ya sea Player, Workstation, o Server) es probable que una vez instalada la aplicaciós pida unas ciertas dependencias para compilar los móos:  

{% img center http://2.bp.blogspot.com/_2-x1cSWxK2M/TR5rKIgQq0I/AAAAAAAAL7M/AxDGVOaXijc/s1600/vmware_req.png 320 320 %}

En OpenSuse es tan facil ejecutar desde un terminal:

```
zypper in gcc kernel-source kernel-syms
```

Saludos


