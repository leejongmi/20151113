#### exercise 1
   - 주기적으로 증가하는 데이터를 openTSDB에 저장

```sh
vim test.py
```

```sh

#!/usr/bin/python

import sys
import urllib2
import time
from datetime import datetime, timedelta
import json
import requests

url ="http://127.0.0.1:4242/api/put"

def insert(value):
        data={
                "metric":"foo.bar",
                "timestamp":time.time(),
                "value":value,
                "tags":{
                        "host":"mypc"
                }
        }
        ret = requests.post(url, data=json.dumps(data))
        print ret
        time.sleep(1)

while 1 :
        t = time.localtime()
        tsec = t.tm_sec

        if tsec%10!=0 :
                print tsec
                time.sleep(1)
        else :
                insert(1111)	
```

```sh

성공
200 - 클라이언트의 요청을 정상적으로 수행하였을때 사용합니다. 응답 바디(body)엔 요청과 관련된 내용을 넣어줍니다. 그리고 200의 응답 바디에 오류 내용을 전송하는데 사용해서는 안된다고 합니다. 오류가 났을땐 40x 응답 코드를 권장합니다.
201 - 클라이언트가 어떤 리소스 생성을 요청하였고, 해당 리소스가 성공적으로 생성되었을때 사용합니다.
202 - 클라이언트의 요청이 비동기적으로 처리될때 사용합니다. 응답 바디에 처리되기까지의 시간 등의 정보를 넣어주면 좋다고 합니다.
204 - 클라이언트의 요청응 정상적으로 수행하였을때 사용합니다. 200과 다른점은 204는 응답 바디가 없을때 사용합니다. 예를들어 DELETE와 같은 요청시에 사용합니다. 클라이언트의 리소스 삭제요청이 성공했지만 부가적으로 응답 바디에 넣어서 알려줄만한 정보가 하나도 없을땐 204를 사용합니다.

실패
400 - 클라이언트의 요청이 부적절할때 사용합니다. 요청 실패시 가장 많이 사용될 상태코드로 예를들어 클라이언트에서 보낸 것들이 서버에서 유효성 검증(validation)을 통과하지 못하였을때 400으로 응답합니다. 응답 바디에 요청이 실패한 이유를 넣어줘야 합니다.
401 - 클라이언트가 인증되지 않은 상태에서 보호된 리소스를 요청했을때 사용하는 요청입니다. 예를들어 로그인(login)하지 않은 사용자가 로그인했을때에만 요청 가능한 리소스를 요청했을때 401을 응답합니다.
403 - 사용자 인증상태와 관계 없이 응답하고싶지 않은 리소스를 클라이언트가 요청했을때 사용합니다. 그러나 해당 응답코드 대신 400을 사용할 것을 권고합니다. 그 이유는 일단 403 응답이 왔다는것 자체는 해당 리소스가 존재한다는 뜻입니다. 응답하고싶지 않은 리소스는 존재 여부 조차 감추는게 보안상 좋기때문에 403을 응답해야할 요청에 대해선 그냥 400이나 404를 응답하는게 좋겠습니다.
404 - 클라이언트가 요청한 리소스가 존재 하지 않을때 사용하는 응답입니다.
405 - 클라이언트가 요청한 리소스에서는 사용 불가능한 Method를 이용했을때 사용하는 응답입니다. 예를들어 읽기전용 리소스에 DELETE Method를 사용했을때 405 응답을 하면 됩니다.

기타
301 - 클라이언트가 요청한 리소스에 대한 URI가 변경 되었을때 사용합니다. 응답시 Location header에 변경된 URI를 적어줘야 합니다.
500 - 서버에 뭔가 문제가 있을때 사용합니다.

```

#### exercise 1 증가
```sh
#!/usr/bin/python

import sys
import urllib2
import time
from datetime import datetime, timedelta
import json
import requests

count = 0

url ="http://127.0.0.1:4242/api/put"

def insert(count):
        data={
                "metric":"foo.bar",
                "timestamp":time.time(),
                "value":count,
                "tags":{
                        "host":"mypc"
                }
        }
        ret = requests.post(url, data=json.dumps(data))
        print ret
        time.sleep(1)

while 1 :
        t = time.localtime()
        tsec = t.tm_sec

        if tsec%10!=0 :
                print tsec
                time.sleep(1)
        else :
		count = count + 10
		insert(count)
```

   
#### exercise 2
   - 주기적으로 특정 웹페이지(http://www.airkorea.or.kr) 크롤링하여 openTSDB에 저장 (미세먼지)
   - ex1 번의 json 데이터를 추가하여 작성..metric은 변경하여 사용하길 바라(참고:dust)
   - url 중복 확인, split 한 변수가 string이므로 int로 형변환 int(c[1])

##### 참고
```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

import urllib2 # extensible library for opening URLs
import time

# 인천 미세먼지 
url = 'http://www.airkorea.or.kr/index'


def getData(buffers):
	a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
	#print a
	
	b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
	#print b

	c = b.split('<td>')
	#print c[1]
	#print c[2]

if __name__ == '__main__':

	page = urllib2.urlopen(url).read()
	print page

	getData(page)

```

#### exercise 2 결과
```sh
#!/usr/bin/python
# -*- coding: utf-8 -*- 

import urllib2 # extensible library for opening URLs
import time
import json
import requests

url = 'http://www.airkorea.or.kr/index'
url_local ="http://127.0.0.1:4242/api/put"

def insert(value):
        data={
                "metric":"dust",
                "timestamp":time.time(),
                "value":value,
                "tags":{
                        "host":"mypc"
                }
        }
        ret = requests.post(url_local, data=json.dumps(data))
        print ret

def getData(buffers):
    a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
    #print a

    b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
    #print b

    c = b.split('<td>')
    #print c[9]
    #print c[10]

    insert(int(c[8]))

if __name__ == '__main__':

    page = urllib2.urlopen(url).read()
    #print page

    getData(page)
```

#### 10초마다 찍어내기
```sh
#!/usr/bin/python
# -*- coding: utf-8 -*- 

import urllib2 # extensible library for opening URLs
import time
import json
import requests

url = 'http://www.airkorea.or.kr/index'
url_local ="http://127.0.0.1:4242/api/put"



def insert(value):
        data={
                "metric":"dust",
                "timestamp":time.time(),
                "value":value,
                "tags":{
                        "host":"mypc"
                }
        }
        ret = requests.post(url_local, data=json.dumps(data))
        print ret


def getData(buffers):
    a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
    #print a

    b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
    #print b

    c = b.split('<td>')
    print c[1]
    print c[2]

    insert(int(c[8]))
if __name__ == '__main__':


    while 1 :
            t = time.localtime()
            tsec = t.tm_sec

            if tsec%10!=0 :
                    print tsec
                    time.sleep(1)
            else :
                    page = urllib2.urlopen(url).read()
                    getData(page)
                    #print page
```
