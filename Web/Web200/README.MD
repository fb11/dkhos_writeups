# Kımızım Kandan Bahtım Karadan - Web 200


```
http://54.89.176.73
```
![](site.png)

Kaynak kodu olarak site şöyle idi ;
``` html
not found
<!-- /site/ -->
```
Hemen `/site/` dizinine gittik.

![](siteIn.png)

Siteyi inceledikten sonra sahte upload yeri ve footerda yazan

```Copyright © PELINCORP - development version: dev``` yazıyordu.

``` html
<p class="copyright text-muted">Copyright &copy; PELINCORP - development version: <a href="http://dev">dev</a></p>
```

Tıkladığımızda ise bizi ``` http://dev```'e atıyordu

![](dev.png)

Bizde eski requestte host paremetresini dev yapıp yolladık

![](host.png)

Host dediğimizde bize dönüyordu.  `/file?name=` parametresi ile dosya okuduğunu gördük ve hemen üstün zekalı gibi flag yazdık.

![](acDe.png)
ve tabiki flagımız ```access denied``` değil. O yüzden enumurate etmeye başladık Web500 de olan server dosyası ismini denedik yani `server.py`.

![](serverpy.png)

``` python
import os
from flask import Flask, request
app = Flask(__name__)

blist = (
    'proc',
    'www',
    'var',
    'etc',
    'root',
    'home',
    'self',
    'flag'
)

def get_flag():
    return os.environ.get('FLAG', '')

def super_firewall(path):
    for b in blist:
        if b in path:
            return False
    return True

@app.route("/")
def hello():
    return "internal file storage, /file?name="

@app.route("/file")
def get_file():
    name = request.args.get('name', 'server.py')
    if not name:
        name = 'server.py'

    status = super_firewall(name)
    if not status:
        return "access denied"

    with open(name, 'r') as fp:
        content = fp.read()
    return content

if __name__ == '__main__':
    app.run(debug=False)
```

Bu şu anlama geliyordu
```
    'proc','www','var','etc','root','home','self','flag'
```
Bu kelimeleri içermeyen tüm sorgular serbest. Flag de zaten `/proc/environ` da duruyor fakat `proc`'u yasaklamışlar. O yüzden `/dev/fd/pid/environ` yolunu izleyecektik. Fakat pid numarası hakkında fikrimiz yoktu o yüzden Intruder ile bruteforce yaparken  regexp ile şansımızı deneyelim dedik (`/dev/fd/../environ`)

![](flag.png)

```
LANG=en_US.UTF-8 SUPERVISOR_SERVER_URL=unix:///var/run/supervisor.sock SUPERVISOR_ENABLED=1 SUPERVISOR_PROCESS_NAME=internal FLAG=DKHOS_{y0u_g0t_m3_pyth0n1st4} SUPERVISOR_GROUP_NAME=internal PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin ```

Yani flagimiz;

 ```
DKHOS_{y0u_g0t_m3_pyth0n1st4}
 ```
