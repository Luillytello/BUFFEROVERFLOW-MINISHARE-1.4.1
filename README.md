# BUFFEROVERFLOW-MINISHARE-1.4.1
***
Se procedera a realizar un desdoblamiento de buffer con el software MINISHARE 1.4.1 habrimos una ventana emergente al lado de nuestro tilix para poder trabajar, usamos el siguiente comando, colacando la IP de la maquina vulnerable.
***
``` 
rdesktop " IP 192.168.a.b"  -u user -p paswword
```
****
### Screenshot
![inicio](https://user-images.githubusercontent.com/104048850/180593311-1796279a-4a4f-4ee8-8fae-baa47898a6e4.png)
Procedemos a realizar un escaneo de vulnerabilidades 
``` 
nmap 192.168.72.135 -sV

```
***
Podemos observar los puertos de la maquina vulnerable sin ejecutar el programa 
***
### Screenshot
![nmap](https://user-images.githubusercontent.com/104048850/180593535-955c4085-6369-4afb-b5e7-dd56e463c24f.png)


