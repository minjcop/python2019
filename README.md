# 파도 정보 분석

학과 | 학번 | 성명
---- | ---- | ---- 
정보컴퓨터공학부 |201524404 |강민진


## 프로젝트 개요
본인의 프로젝트 개요에 대하여 작성합니다.

## 사용한 공공데이터 
[데이터보기](https://github.com/cybermin/python2019/blob/master/%EB%B6%80%EC%82%B0%EA%B5%90%ED%86%B5%EA%B3%B5%EC%82%AC_%EB%8F%84%EC%8B%9C%EC%B2%A0%EB%8F%84%EC%97%AD%EC%82%AC%EC%A0%95%EB%B3%B4_20190520.csv)
https://data.kma.go.kr/data/sea/selectBuoyRltmList.do?pgmNo=52

## 소스
* [링크로 소스 내용 보기](https://github.com/cybermin/python2019/blob/master/tes.py) 

* 코드 삽입
import pandas as pd
import numpy as np
#import pyplot as plt
from matplotlib import pyplot as plt
#count = 0
# 풍속은 높을수록 안좋음. (30 - (풍속)) * 5
# 80점은 되어야 함.

# x(풍향) 180 넘을 경우에는 100 - (360 - x)
#   "     180 안넘을 경우에는 100 - (x - 0)
# 풍속이 낮으면 풍향도 영향이 거의 없기 때문에
# 풍속이 120점 이상 = 풍향에 50점 추가
# 풍속이 100점 이상 120점 미만 = 풍향에 25점 추가
# 파고는 * 150점
# 80점은 되어야 함

# 파주기 길수록 좋음 * 15점
# 80점은 되어야 함

# 파향은 180이 제일 높은 점수. 180과 해당 파향빼고 절댓값해서 점수
# 100 - abs(180 - x)가 60점은 되어야 함. 80점은 되어야 수월

#함수에 넣어서 구한 점수가 각 80점이 넘어야 괜찮은 파도
#괜찮은 파도인 날만 뽑아서 데이터프레임에 넣어주면 될듯.

def scoreFunc(windspeed, winddirection, waveheight, waveperiod, wavedirection) :

    a = (33 - windspeed) * 5 # 풍속
    if winddirection > 180 :
        b = 120 - abs(360 - winddirection) # 풍향
    else :
        b = 120 - abs(winddirection - 0)
    if a > 120 :
        b = b + 50
    elif a > 100 :
        b = b + 25
    c = waveheight * 150 # 파고
    d = waveperiod  * 15
    e = 130 - abs(180 - wavedirection)

    return a + b + c + d + e

def countFunc(windspeed, winddirection, waveheight, waveperiod, wavedirection) :
    global count
    count = 0
    a = (33 - windspeed) * 5 # 풍속
    if winddirection > 180 :
        b = 120 - abs(360 - winddirection) # 풍향
    else :
        b = 120 - abs(winddirection - 0)
    if a > 120 :
        b = b + 50
    elif a > 100 :
        b = b + 25
    c = waveheight * 150 # 파고
    d = waveperiod  * 15
    e = 130 - abs(180 - wavedirection)
    if a > 80 :
        count = count + 1
    if b > 80 :
        count = count + 1
    if c > 80 :
        count = count + 1
    if d > 80 :
        count = count + 1
    if e > 80 :
        count = count + 1

    return count

df = pd.read_csv('2018다대포2.csv', encoding = 'cp949') #쉼표사이에 파일경로/파일명.csv
df.columns = ['위치', 'date', '풍속', '풍향', '파고', '파주기', '파향']
del df['위치']

windspeed = df.groupby(df.date.str[:10]).mean()['풍속'] # 뒤에 [index]붙여서 각각 표현 가능 0부터 시작.
winddirection = df.groupby(df.date.str[:10]).mean()['풍향']
waveheight = df.groupby(df.date.str[:10]).mean()['파고']
waveperiod = df.groupby(df.date.str[:10]).mean()['파주기']
wavedirection = df.groupby(df.date.str[:10]).mean()['파향']

scoreList = []
countList = []

for i in range(0, 91) :
    scoreList.append(int(scoreFunc(windspeed[i], winddirection[i], waveheight[i], waveperiod[i], wavedirection[i])))
    countList.append(countFunc(windspeed[i], winddirection[i], waveheight[i], waveperiod[i], wavedirection[i]))

scoreIndex = []
countIndex = []

for score in scoreList :
    if score > 530 :
        scoreIndex.append(scoreList.index(score))

#for count in countList :
#    if count >= 4 :
        #countIndex.append(countList.index(count))
#print(countIndex)

for i in range(0, 91) :
    if countList[i] >= 4 :
        countIndex.append(i)

print(scoreIndex)
print(countIndex)

#각 점수 및 count
for index in scoreIndex :
    print(scoreList[index])
for index in countIndex :
    print(countList[index])
#count

t1 = np.arange(0, 91, 1)

plt.figure(1)
plt.title('Wave Point')
plt.plot(t1, countList, c = 'g', lw = 1, ls = ':')
plt.xlabel('day')
plt.ylabel('point( /4)')
plt.legend()
#score

plt.figure(2)
plt.title('Wave Score')
plt.plot(t1, scoreList, c = 'b', lw = 1, ls = '--')
plt.xlabel('day')
plt.ylabel('score( /550)')
plt.legend()
#plt.xlabel('날짜')
#plt.ylabel('count')
plt.show()


    #print(scoreList[scoreIndex[i]])
    #print(countList[countIndex[i]])


#for score in scoreList :
#    if score > 500 :
#        print(score)
#for count in countList :
#    if count >= 4 :
#        print(count)

#print(scoreList)
#print(countList)


#각각 구해졌으니 새로운 column을 생성해서 거기에 함수의 리턴값을 저장하면 됨.
# 1. 함수 구현 완료, 2. column 생성, 3. 그래프.


#풍속, 풍향, 파고 ... 각각을 column으로 나눴다.
#나눈 것을 함수에 대입해서 점수를 리턴받는다
#그 점수가 모두 80점이 넘는 것들을 구하고
#그것들을 더한 점수를 출력해서 그래프로 만들어주면
#그 점수에 따라 적합한 파도가 서열순대로 나뉠것임.
#df.groupby(df.date.str[:10]).mean()
