---
layout: post
title: "[Crawler] 크롤링 Class"
categories: posts
tags: crawler
---

# [Crawler] class 형식으로 Crawler만들기


```
# 설정 및 패키지 설치

pip install beautifulsoup4 lxml
pip install requests

# 폴더 생성
crawler / 
data /             <- 패키지 폴더(class 깔끔하게 관리하기)
__init__.py
episode.py
webtoon.py
crawler.py
```

### crawler.py

```
import os
import pickle
import re

import requests
from bs4 import BeautifulSoup

from data import Episode, Webtoon, WebtoonNotExist


class Crawler:
ROOT_PATH = os.path.dirname(os.path.abspath(__file__))
SAVE_PATH = os.path.join(ROOT_PATH, 'saved_data')

def __init__(self):
self._webtoon_dict = {}
# 초기화 함수에서 저장 폴더를 만들어줌
os.makedirs(self.SAVE_PATH, exist_ok=True)

def get_html(self):
"""
전체 웹툰 목록의 HTML을 리턴한다.
만약에 파일로 저장되어 있다면, 해당 내용을 읽어온다.

파일로 저장되어 있지 않다면, requests를 사용해 웹에서 받아와 리턴해준다.
파일위치는 ./saved_data/weekday.html를 사용
경로 작성에는 os.path모듈을 사용

-> 다 작성한 후에는 show_webtoon_list에서 이 메서드를 사용하도록 함
:return: HTML데이터 문자열
"""
# weekday.html파일의 경로
root = os.path.dirname(os.path.abspath(__file__))
dir_path = os.path.join(root, 'saved_data')
file_path = os.path.join(dir_path, 'weekday.html')

if os.path.exists(file_path):
# 경로가 존재하면
# 해당 경로의 파일을 읽기
html = open(file_path, 'rt').read()
else:
# 경로가 존재하지 않으면
# HTTP요청결과를 가져오고, 이후 다시 요청시 읽기위한 파일을 기록

# 파일을 저장할 폴더를 생성. 이미 존재하는 경우 무시하기 위해 exist_ok인수 추가
os.makedirs(dir_path, exist_ok=True)
response = requests.get('https://comic.naver.com/webtoon/weekday.nhn')
html = response.text
open(file_path, 'wt').write(html)
return html

@property
def webtoon_dict(self):
if not self._webtoon_dict:
html = self.get_html()
soup = BeautifulSoup(html, 'lxml')
col_list = soup.select_one('div.list_area.daily_all').select('.col')
li_list = []
for col in col_list:
col_li_list = col.select('.col_inner ul > li')
li_list.extend(col_li_list)

for li in li_list:
href = li.select_one('a.title')['href']
m = re.search(r'titleId=(\d+)', href)
webtoon_id = m.group(1)
title = li.select_one('a.title').get_text(strip=True)
url_thumbnail = li.select_one('.thumb > a > img')['src']

if title not in self._webtoon_dict:
new_webtoon = Webtoon(webtoon_id, title, url_thumbnail)
self._webtoon_dict[title] = new_webtoon
return self._webtoon_dict

def get_webtoon(self, title):
"""
title이 제목인 Webtoon객체를 가져옴

:param title:
:return:
"""
try:
return self.webtoon_dict[title]
except KeyError:
raise WebtoonNotExist(title)

def show_webtoon_list(self):
for title, webtoon in self.webtoon_dict.items():
print(title)

def save(self):
# self._webtoon_dict객체를
# ./saved_data/cralwer.pickle 파일에 기록
with open(os.path.join(self.SAVE_PATH, 'crawler.pickle'), 'wb') as f:
pickle.dump(self._webtoon_dict, f)

def load(self):
# ./saved_data/crawler.pickle파일의 내용을 가지고
# self._webtoon_dict객체를 재구성
with open(os.path.join(self.SAVE_PATH, 'crawler.pickle'), 'rb') as f:
self._webtoon_dict = pickle.load(f)


if __name__ == '__main__':
crawler = Crawler()
w = crawler.get_webtoon('신의 탑')
print(w.episode_dict)
```

### episode.py

```
class Episode:
def __init__(self, episode_id, title, url_thumbnail, rating, created_date):
self.episode_id = episode_id
self.title = title
self.url_thumbnail = url_thumbnail
self.rating = rating
self.created_date = created_date

def __repr__(self):
return self.title
```


### webtoon.py

+ all
+ * 로 모듈을 불러올때 all을 쓰면 지정한 클래스만 불러옴

```
import re
from collections import OrderedDict

import requests
from bs4 import BeautifulSoup

from . import Episode

__all__ = (
'Webtoon',
'WebtoonNotExist',
)


class Webtoon:
TEST_WEBTOON_ID = 714834
EPISODE_LIST_BASE_URL = 'https://comic.naver.com/webtoon/list.nhn'

def __init__(self, webtoon_id, title, url_thumbnail):
self.webtoon_id = webtoon_id
self.title = title
self.url_thumbnail = url_thumbnail
self._episode_dict = OrderedDict()

def __repr__(self):
return self.title

def get_episode_list_url_params_dict(self, **kwargs):
# 기본적으로 {'titleId': <자신의 webtoon_id>}
#  인 dict를 리턴
# 키워드인자가 주어지면 해당 키워드인자를 추가한 dict를 리턴
# ex) 그냥 호출시 리턴: {'titleId': 234343}
#     get_epi..(page=1)로 호출시 리턴:
#       {'titleId': 234234, 'page': 1}
params = {'titleId': self.webtoon_id}
params.update(kwargs)
return params

@property
def episode_dict(self):
"""
에피소드 목록을 파싱하는부분을 먼저 연습
-> notebook에서

자신의 _episode_dict가 있다면 그대로 리턴
아니라면 채워준다
주소는 (에피소드 리스트 URL)
https://comic.naver.com/webtoon/list.nhn?titleId={self.webtoon_id}

키는 Episode의 episode_id
에피소드 상세 URL에서의 'no' GET parameter값에 해당
https://comic.naver.com/webtoon/detail.nhn?titleId=651673&no=345&weekday=wed

webtoon_dict를 채울때와 비슷하게, 새로운 Episode클래스 인스턴스를 만들어 할당

1. 현재는 1페이지 내용만 불러오므로
page값을 1부터 늘려가면서 '다음'버튼이 안보일때까지 내용을 가져옴 (페이지수가 적은 웹툰으로 테스트!)
2. pickle을 사용해서 Crawler가 가진 webtoon_dict를 저장, 불러오기 하는 방식으로 중복 데이터를 웹에서 받지 않도록 함
3. CLI를 구성해서 사용자가 셸에서 선택해서 웹툰 크롤러 기능을 사용할 수 있도록 만들기 (main.py)
4. Episode의 Detail페이지에서 그림을 다운로드 받기
requests로 그림 요청시 Referer설정을 해줘야 함 <- 아니면 403또는 400에러 발생
headers = {'Referer': 'http://comic.naver.com/webtoon/list.nhn?titleId=<WebtoonID>'}
requests.get(<URL>, headers=headers)

저장시
response = requests.get(<URL>)
open(<path>, 'wb').write(response.content)
코드를 사용
5. 다운로드 받은 그림을 볼 수 있는 HTML생성하기

:return:
"""
def get_page_episode_dict(page):
"""
:param page: 크롤링 할 페이지
:return: {
'has_next': <다음페이지가 있는지>',
'episode_dict': 'episode_id를 키, Episode인스턴스를 값으로 쓰는 dict
}
"""
# 위에서 만든 메서드에서 리턴된 dict를 사용,
# requests를 이용한 요청에 GET parameters를 전달
response = requests.get(
self.EPISODE_LIST_BASE_URL,
params=self.get_episode_list_url_params_dict(page=page)
)
soup = BeautifulSoup(response.text, 'lxml')
# 이 page에 해당하는 Episode들을 담을 dict
page_episode_dict = OrderedDict()

# 에피소드 목록이 table.viewList의 각 'tr'요소 하나씩에 해당함
table = soup.select_one('table.viewList')
tr_list = table.select('tr')[1:]
for tr in tr_list:
# 이 루프 내부의 'tr'하나당 에피소드 하나를 만들어야 함
try:
# 데이터 파싱
td_list = tr.select('td')
href = td_list[0].select_one('a')['href']
no = re.search(r'no=(\d+)', href).group(1)
url_thumbnail = td_list[0].select_one('img')['src']
title = td_list[1].select_one('a').get_text(strip=True)
rating = td_list[2].select_one('strong').get_text()
created_date = td_list[3].get_text(strip=True)
# 파싱한 데이터로 새 Episode인스턴스 생성
episode = Episode(
episode_id=no,
title=title,
url_thumbnail=url_thumbnail,
rating=rating,
created_date=created_date,
)
# 리턴해줄 dict변수에 값 할당
page_episode_dict[no] = episode
except:
# 위 파싱에 실패하는 경우에는 무시 (tr이 의도와 다르게 생겼을때 실패함)
# 기왕이면 실패 로그를 쌓으면 좋음 (어떤 웹툰의 몇 번째 페이지 몇 번째 row시도중 실패했다를 텍스트 파일에)
pass

next_btn = soup.select_one('.paginate a.next')
return {
'episode_dict': page_episode_dict,
'has_next': bool(next_btn),
}

if not self._episode_dict:
# 비어있는 경우
# 1페이지부터 끝페이지까지 get_page_episode_dict를 실행한 결과를 self._episode_dict에 추가
page = 1
while True:
# while문을 반복하며 증가되는 page값을 사용해서 크롤링한 결과 dict및 다음 페이지 존재여부를 가져옴
result = get_page_episode_dict(page)
page_episode_dict = result['episode_dict']
# 결과 dict를 인스턴스의 _episode_dict에 업데이트
self._episode_dict.update(page_episode_dict)
# 다음 loop에서 증가한 page값을 사용하기 위해 값 설정
page += 1

# 결과에서 다음 페이지 존재여부를 판단, 없다면 break로 반복문 탈출
has_next = result['has_next']
if not has_next:
break
print(f'== {self.title}의 에피소드 목록을 크롤링 ==')
return self._episode_dict

def get_episode(self, index):
"""
index번에 해당하는 에피소드를 자신의 episode_dict프로퍼티를 사용해서 리턴
:param index:
:return:
"""
pass


class WebtoonNotExist(Exception):
def __init__(self, title):
self.title = title

def __str__(self):
return f'Webtoon(title: {self.title})을 찾을 수 없습니다'

```
