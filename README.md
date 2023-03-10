- π μ λͺ©: μ°ν¬λΌμ΄λ-λ¬μμ μ μμ μΈλͺμμ€κ³Ό μ₯λΉμμ€μ λν λ°μ΄ν°μκ°ν
- π± μ±λͺ: μ μ°¬νΈ, κ΅λ²: 19766
- π νλ‘μ νΈ λͺ©μ : λ¬μμμ μμ€μ κΈ°μ€μΌλ‘ λ°μ΄ν°λ₯Ό μκ°ννκ³  λΆμνλ©΄μ μ μ₯μ μν©μ λν΄ λ³΄λ€ μμΈν μκΈ° μν¨
-------------------------------------------------------------------------------------------------------
- κ΄λ ¨ μλ£ λ°μ΄ν° λ§ν¬ <https://www.kaggle.com/datasets/piterfm/2022-ukraine-russian-war>
- μ¬μ©λ νλ‘κ·Έλ¨
- Python - 2022 Ukraine Russia War: visualization
- R - Losses of Russia (Visualization)
- Data Daily + Plot Example
-------------------------------------------------------------------------------------------------------
- μ½λμ€λͺ(κ΅¬μ²΄μ μΈ μ½λ μ€λͺμ hwpνκΈνμΌμ μ μ΄λμμΌλ, ν΅μ¬μ μΈ μ½λλ₯Ό μ€μ¬μΌλ‘ μ€λͺνκ² λ€.)

##μ΄λ μ½νΈλ₯Ό ν΅ν΄ λΌμ΄λΈλ¬λ¦¬ μ€μ μ€μ

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

##μ½λ©μ csv νμΌμ μ°κ²°

from google.colab import files

uploaded = files.upload()

---------------------------------------------------------------------------------------------------

##csvνμΌ μ°κ²°

equipment = pd.read_csv('/content/russia_losses_equipment.csv')

personnel = pd.read_csv('/content/russia_losses_personnel.csv')

---------------------------------------------------------------------------------------------------

##mssingno λ§€νΈλ¦­μ€λ‘ κ²°μΈ‘μΉ νμ

msno.matrix(equipment)
---------------------------------------------------------------------------------------------------

##pie κ·Έλνλ‘ μΈλͺμμ€, ν¬λ‘μμ€ λ°μ΄ν° μ

personnel_ = personnel[['personnel', 'POW']].max()

fig = px.pie(names = personnel_.index,

values = personnel_.values,

title = 'Personnel Losses and POWs')
             
fig.update_traces(textposition='inside', textinfo='value+percent+label')

fig.show()

msno.matrix(equipment)

---------------------------------------------------------------------------------------------------

##μ ν κ·Έλνλ‘ μΈλͺμμ€, ν¬λ‘ μμ€ λ°μ΄ν° μκ°ν

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

##μ₯λΉμμ€ κ΄λ ¨ λ°μ΄ν° μκ°ν

#cummulative loss of equipment

equipment_ = equipment[equipment.columns[2:]].iloc[-1]#.sum(numeric_only=True)

fig = px.pie(values=equipment_.values,

names=equipment_.index,
             
title='Total number of losses by equipment')
             
fig.update_traces(textposition='inside', textinfo='percent+label')

fig.show()

---------------------------------------------------------------------------------------------------

##μ¬νλ pie κ·Έλνλ‘ μ₯λΉ μμ€ νμ(μ‘.ν΄.κ³΅ μ€μ¬μΌλ‘)

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

##μ μ²΄ μμ€ νν© νμ κ·Έλν 

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

##μκ΄κ΄κ³ κ³μ νμ κ·Έλν

plt.figure(figsize=(12,12))

sns.heatmap(personnel.merge(equipment, left_on=['date','day'],

right_on=['date','day']).corr(),
                                                   
annot=True, cmap='Blues')

plt.show()

---------------------------------------------------------------------------------------------------

##νμ€ν κ·Έλ¨μ ν΅ν μ μ²΄ νΌν΄μ§μ­μ λ°λ₯Έ μμ€ κ·Έλν

fig = px.histogram(equipment['greatest losses direction'][89:])

fig.update_xaxes(tickangle=40)

fig.show()

------------------------------------------------------------------------------------------------

<νκ³μ  λ° λ¬Έμ μ  λΆμ>: λ¨Όμ  νκ³μ μΌλ‘λ κ°μ₯ κΈ°λ³Έμ μΌλ‘ μ μ₯μμ μΌμ΄λλ νΌν΄μ λν λ°μ΄ν° μμ§μ΄ κ°μ₯ μ΄λ ΅λ€. κ΅μλκ»μ μμ μκ°μ λ§μνμ¨λ―μ΄ μ¬μ€ μΈκ³΅μ§λ₯ νμ΅μ κ°μ₯ μ€
μν κ²μ κ·Έκ²μ κΈ°λ°μΌλ‘νλ λ°μ΄ν°μμ΄λ€. νμ§λ§ μ μ₯μ΄λΌλ νΉμμ±μ μ°λ¦¬κ° μ»μ΄μΌνλ μ μ₯ λ°μ΄ν°λ₯Ό μ»κΈ° μ½μ§ μμ νκ²½μμ λκ΅¬λ μ κ²μ΄λ€. μ΄κ²μ΄ λ΄κ° μ°Ύμ κ°μ₯ ν° νκ³μ μ΄λΌ 
μκ°νλ€. μΆκ°μ μΈ λ¬Έμ μ μ μμ€μ λ²μκ° κ΅μ₯ν μ λ§€νλ€λ κ²μ΄λ€. κ°λ Ή special equipment(νΉμμ₯λΉ)μ λ²μλ₯Ό μ΄λ μ λλ‘ μ νν  κ²μΈκ°μ λν ν΅μΌμμ΄ μ‘΄μ¬νμ§ μλλ€. κ·Έλ κΈ° λ
λ¬Έμ λ°μ΄ν°λ₯Ό μμ§νλ μ¬λλ€μ μ£Όκ΄μ μΈ μ¬κ³ κ° μν₯μ λ―ΈμΉκ² λκ³ , μ΄κ²μ λΆμμ ν λ°μ΄ν°λ₯Ό μ»μ νλ₯ μ λμΌ κ²μ΄λ€. μΆκ°λ‘ νΉμμ₯λΉμ λ²μλ₯Ό λκ² μ‘μμ λ λ€λ₯Έ ν­λͺ©μ μ₯λΉκΉμ§ ν¬
ν¨ν  μ μλ€λ λ¬Έμ μ  λν μ‘΄μ¬ν  κ²μ΄λ€.

--------------------------------------------------------------------------------------------------

<νκ΅­κ΅° μ μ© λ°©μ>: λ¨Όμ  λλ μ΄ μκ³ λ¦¬μ¦μ νκ΅­μ μμ μ μ©ν΄λ³΄κ³  μΆλ€λ λ§μμ΄ νλ‘μ νΈ μμ λλΆν° μ»Έλ€. μ¬μ€ νκ΅­κ΅°μ μ λ³΄λ₯Ό ν λλ‘ μκ³ λ¦¬μ¦μ λ³κ²½νλ κ²μ΄ μ²μμ λͺ©νμμ§λ§ ν
κ΅­κ΅°μ λ μ§λ³ μΈλͺ λ° μ₯λΉμμ€μ μ νν νμν  μ μλ μλ£λ νμ¬ μ‘΄μ¬νμ§ μμλ€. νμ§λ§ λλ΅μ μΈ μμ€ νΌν΄νν©μ μ μ μλ€λ©΄ κ·Έκ²μ ν΅ν΄ λ μ§λ³ μμ€νν©μ μμΈ‘ν λ€ μ΄λ₯Ό μ μ©ν
λ€λ©΄ μ°λ¦¬λ νλ¦¬ν°κ° λμ νκ΅­μ μμ ν΄μμ ν  μ μμ κ²μμ νμ νλ€. 

----------------------------------------------------------------------------------------------------


<μ₯λ³ νμ© λ°©μ>: λ΄κ° μκ°νμ λ μ΄ νλ‘μ νΈλ κ΅³μ΄ μ μμμλ§ μ°μΌ νμκ° μλ€. μ₯λ³λ€μ΄ νλ ¨ν  λλ μΈλͺμμ€κ³Ό μ₯λΉμμ€μ λΆλͺν λ°μνλ€. νΉν KCTCνλ ¨κ³Ό κ°μ΄ μ€μ  μ μ₯μ κ°μ 
νκ³  μ€μνλ νλ ¨λ€μμλ μ¬ν νκ°λ₯Ό ν  λ νλ ¨ κ°μ μΈλͺμμ€ λ° μ₯λΉμμ€μ νμνλ€λ©΄ νλ ¨μ μ§μ΄ ν¨μ¬ λμμ§ κ²μΌλ‘ μμνλ€.
