- 👋 제목: 우크라이나-러시아 전쟁의 인명손실과 장비손실에 대한 데이터시각화
- 🌱 성명: 정찬호, 교번: 19766
- 👀 프로젝트 목적: 러시아의 손실을 기준으로 데이터를 시각화하고 분석하면서 전장의 상황에 대해 보다 자세히 알기 위함
-------------------------------------------------------------------------------------------------------
- 관련 자료 데이터 링크 <https://www.kaggle.com/datasets/piterfm/2022-ukraine-russian-war>
- 사용된 프로그램
- Python - 2022 Ukraine Russia War: visualization
- R - Losses of Russia (Visualization)
- Data Daily + Plot Example
-------------------------------------------------------------------------------------------------------
- 코드설명(구체적인 코드 설명은 hwp한글파일에 적어놓았으니, 핵심적인 코드를 중심으로 설명하겠다.)

##어래 코트를 통해 라이브러리 설정실시

import libraries

import pandas as pd

import numpy as np

import matplotlib.pyplot as plt

import seaborn as sns

import missingno as msno

import plotly

import plotly.graph_objs as go

from plotly.subplots import make_subplots

import plotly.express as px

---------------------------------------------------------------------------------------------------

##코랩에 csv 파일을 연결

from google.colab import files

uploaded = files.upload()

---------------------------------------------------------------------------------------------------

##csv파일 연결

equipment = pd.read_csv('/content/russia_losses_equipment.csv')

personnel = pd.read_csv('/content/russia_losses_personnel.csv')

---------------------------------------------------------------------------------------------------

##mssingno 매트릭스로 결측치 파악

msno.matrix(equipment)
---------------------------------------------------------------------------------------------------

##pie 그래프로 인명손실, 포로손실 데이터 시

personnel_ = personnel[['personnel', 'POW']].max()

fig = px.pie(names = personnel_.index,

values = personnel_.values,

title = 'Personnel Losses and POWs')
             
fig.update_traces(textposition='inside', textinfo='value+percent+label')

fig.show()

msno.matrix(equipment)

---------------------------------------------------------------------------------------------------

##선형 그래프로 인명손실, 포로 손실 데이터 시각화

fig = make_subplots(specs=[[{"secondary_y": True}]])

fig.add_trace(go.Scatter(x=personnel.date,

 y=personnel.personnel,
                         
 name='Personnel',
                         
line=dict(color='blue')))
                         
fig.add_trace(go.Scatter(x=personnel.date,

y=personnel.POW,
                         
name='POW',
                         
line=dict(color='red')),
                         
 secondary_y=True)
                         
fig.update_layout(title='Personnel Losses',

yaxis=dict(title='Personnel',
                  
tickfont=dict(color="blue"),
                             
titlefont=dict(color="blue")),
                             
 yaxis2=dict(title='POW',
                  
titlefont=dict(color="red"),
                             
tickfont=dict(color="red")))
                             
fig.show()

---------------------------------------------------------------------------------------------------

##장비손실 관련 데이터 시각화

#cummulative loss of equipment

equipment_ = equipment[equipment.columns[2:]].iloc[-1]#.sum(numeric_only=True)

fig = px.pie(values=equipment_.values,

names=equipment_.index,
             
title='Total number of losses by equipment')
             
fig.update_traces(textposition='inside', textinfo='percent+label')

fig.show()

---------------------------------------------------------------------------------------------------

##심화된 pie 그래프로 장비 손실 파악(육.해.공 중심으로)

equipment_ = equipment[equipment.columns[2:]].select_dtypes(include=np.number).iloc[-1]#.sum(numeric_only=True)

equipment_ = equipment_.reset_index()

equipment_.columns = ['equipment', 'count']

air_units = ['drone','aircraft','helicopter', 'cruise missiles']

naval_units = ['naval ship']

def get_type(equipment):

if equipment in air_units:
    
return 'Air'
        
if equipment in naval_units:
    
return 'Naval'
        
else:
    
return 'Ground'

equipment_['type'] = equipment_['equipment'].apply( lambda x: get_type(x))

fig = px.sunburst(equipment_, path=['type', 'equipment'], values='count',

title='Equipment losses grouped by type')

fig.show()

---------------------------------------------------------------------------------------------------

##전체 손실 현황 파악 그래프 

fig = go.Figure()

fig.add_trace(go.Scatter(x=equipment.date, 

 y=equipment[equipment.columns[2:]].sum(
                         
axis=1, numeric_only=True),
                         
mode='lines+markers'))
                         
fig.update_layout(title='All Units Losses')

fig.show()

air_units = ['drone','aircraft','helicopter', 'cruise missiles']

naval_units = ['naval ship']

ground_units = [x for x in equipment.columns if x not in air_units+naval_units+['date','day']]

fig = go.Figure()

for column in air_units:

fig.add_traces(go.Scatter(x=equipment.date,
    
y=equipment[column],
                              
name=column,
                              
 mode='lines+markers'))
                              
fig.update_layout(title='Air Units Losses')

fig.show()

fig = go.Figure()

for column in ground_units:

fig.add_traces(go.Scatter(x=equipment.date,
    
y=equipment[column],
                              
name=column,
                              
 mode='lines+markers'))
                              
fig.update_layout(title='Ground Units Losses')

fig.show()

fig = go.Figure()

for column in naval_units:

fig.add_traces(go.Scatter(x=equipment.date,
 
y=equipment[column],
                              
name=column,
                              
 mode='lines+markers'))
                              
fig.update_layout(title='Naval Units Losses - naval ships')

fig.show()

---------------------------------------------------------------------------------------------------

##상관관계 계수 파악 그래프

plt.figure(figsize=(12,12))

sns.heatmap(personnel.merge(equipment, left_on=['date','day'],

right_on=['date','day']).corr(),
                                                   
annot=True, cmap='Blues')

plt.show()

---------------------------------------------------------------------------------------------------

##히스토그램을 통한 전체 피해지역에 따른 손실 그래프

fig = px.histogram(equipment['greatest losses direction'][89:])

fig.update_xaxes(tickangle=40)

fig.show()

------------------------------------------------------------------------------------------------

<한계점 및 문제점 분석>: 먼저 한계점으로는 가장 기본적으로 전장에서 일어나는 피해에 대한 데이터 수집이 가장 어렵다. 교수님께서 수업 시간에 말씀하셨듯이 사실 인공지능 학습에 가장 중
요한 것은 그것을 기반으로하는 데이터셋이다. 하지만 전장이라는 특수성은 우리가 얻어야하는 전장 데이터를 얻기 쉽지 않은 환경임은 누구나 알 것이다. 이것이 내가 찾은 가장 큰 한계점이라 
생각한다. 추가적인 문제점은 손실의 범위가 굉장히 애매하다는 것이다. 가령 special equipment(특수장비)의 범위를 어느 정도로 제한할 것인가에 대한 통일안이 존재하지 않는다. 그렇기 때
문에 데이터를 수집하는 사람들의 주관적인 사고가 영향을 미치게 되고, 이것은 불완전한 데이터를 얻을 확률을 높일 것이다. 추가로 특수장비의 범위를 넓게 잡았을 때 다른 항목에 장비까지 포
함할 수 있다는 문제점 또한 존재할 것이다.

--------------------------------------------------------------------------------------------------

<한국군 적용 방안>: 먼저 나는 이 알고리즘을 한국전쟁에 적용해보고 싶다는 마음이 프로젝트 시작 때부터 컸다. 사실 한국군의 정보를 토대로 알고리즘을 변경하는 것이 처음의 목표였지만 한
국군의 날짜별 인명 및 장비손실을 정확히 파악할 수 있는 자료는 현재 존재하지 않았다. 하지만 대략적인 손실 피해현황을 알 수 있다면 그것을 통해 날짜별 손실현황을 예측한 뒤 이를 적용한
다면 우리는 퀄리티가 높은 한국전쟁의 해석을 할 수 있을 것임에 확신한다. 

----------------------------------------------------------------------------------------------------


<장병 활용 방안>: 내가 생각했을 때 이 프로젝트는 굳이 전쟁에서만 쓰일 필요가 없다. 장병들이 훈련할 때도 인명손실과 장비손실은 분명히 발생한다. 특히 KCTC훈련과 같이 실제 전장을 가정
하고 실시하는 훈련들에서도 사후 평가를 할 때 훈련 간의 인명손실 및 장비손실을 파악한다면 훈련의 질이 훨씬 높아질 것으로 예상한다.
