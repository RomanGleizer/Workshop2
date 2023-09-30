# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #2 выполнил:
- Глейзер Роман Дмитриевич
- РИ-220930

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

## Задание 1
### Выберите одну из компьютерных игр, приведите скриншот её геймплея и краткое описание концепта игры. Выберите одну из игровых переменных в игре (ресурсы, внутри игровая валюта, здоровье персонажей и т.д.), опишите её роль в игре, условия изменения / появления и диапазон допустимых значений. Постройте схему экономической модели в игре и укажите место выбранного ресурса в ней.

### Игра: "Веселая ферма"
![image](https://github.com/RomanGleizer/Workshop2/assets/125725530/aa806a0b-c9a0-4ede-838a-d5ff637fd9a7)

Ход работы:
- Кратко описать концепт игры, выбрать игровую переменную, описать её характеристики и роль в игре и построить экономическую модель игры.

Описание концепта игры.
- "Веселая ферма" - популярная аркадная игра, которая предоставляет игрокам возможность превратить запущенную ферму в большую и развитую, используя игровые ресурсы. 
В игре нужно заниматься разведением скота, продажей товаров, улучшением строений и противостоять медведям.

Рассмотрим одну из игровых переменных в игре.
- Игровая валюта (монеты или звезды). Основной ресурс в игре.
  
- Валюта может использоваться для:
  Покупки семян и ресурсов, животных, птиц, строительных материалов и улучшений для зданий, так же для закупок необходимых предметов и инструментов.

- Изменятся игровая валюта может после осуществления операций. Например, покупки, улучшения.

- Минимальное значение: 0 монет. Это стартовое значение при начале игры, когда у игрока еще нет средств.

- Максимальное значение: В зависимости от механики, максимальное количество монет может достигать нескольких миллионов или более.

Схема экономической модели в игре.
Игровая валюта занимает очень важную роль в модели, поэтому я расположил её в центре.

![Economy](https://github.com/RomanGleizer/Workshop2/assets/125725530/0e1f95e4-7402-4180-bc6d-84dea77981c7)

## Задание 2
### С помощью скрипта на языке Python заполните google-таблицу данными, описывающими выбранную игровую переменную в выбранной игре (в качестве таких переменных может выступать игровая валюта, ресурсы, здоровье и т.д.). Средствами google-sheets визуализируйте данные в google-таблице (постройте график, диаграмму и пр.) для наглядного представления выбранной игровой величины.

Ход работы:
- Настроить доступ к google-таблице по api, написать код генерации данных с помощью Python, используя gspread. 
- Передать данные в таблицу, построить график и диаграмму, используя переданные данные.

```py

import gspread
import numpy as np
gc = gspread.service_account(filename='unitydatascience-400313-4d94d452be4c.json')
sh = gc.open("UnityWorkshop2")
price = np.random.randint(2000, 10000, 11)
mon = list(range(1,11))
i = 0
while i <= len(mon):
    i += 1
    if i == 0:
        continue
    else:
        tempInf = ((price[i-1]-price[i-2])/price[i-2])*100
        tempInf = str(tempInf)
        tempInf = tempInf.replace('.',',')
        sh.sheet1.update(('A' + str(i)), str(i))
        sh.sheet1.update(('B' + str(i)), str(price[i-1]))
        sh.sheet1.update(('C' + str(i)), str(tempInf))
        print(tempInf)

```

![image](https://github.com/RomanGleizer/Workshop2/assets/125725530/22df2ae9-588e-4280-a14b-5f0efab1a3c3)

## Задание 3
### Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной. Например, если выбрано здоровье главного персонажа вы можете выводить сообщения, связанные с его состоянием.

Ход работы:
- Создать ключ API для передачи данных из google-таблицы в Unity.
- Создать новый 3D проект на Unity, добавить необходимые звуковые файлы.
- Написать скрипт для воспроизведения звуков, в зависимости от значения в таблице.


```cs

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;
using System;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string,float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    void Awake()
    {
        StartCoroutine(GoogleSheets());
    }

    void Update()
    {
        if (!dataSet.ContainsKey("Mon_" + i.ToString())) return;

        if (dataSet["Mon_" + i.ToString()] <= 10 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log("Инфляция: " + dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 10 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log("Инфляция: " + dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count + 1)
        {
            StartCoroutine(PlaySelectAudioBad());
            Debug.Log("Инфляция: " + dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get(
            "https://sheets.googleapis.com/v4/spreadsheets/1cY1tcBfyk_0_Dw7KC-Ss8yxl0i3GmMu3qQDhc37nxTo/values/A1%3AZ100?key=AIzaSyC38tVCRjiGN95fs0N4bcfGWqgnaMJoA5g");

        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;
            dataSet.Add("Mon_" + selectRow[0], float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

![image](https://github.com/RomanGleizer/Workshop2/assets/125725530/b717b276-92d3-4c32-a387-c28a04197427)

## Выводы

Я узнал как передавать данные в гугл таблицу, используя язык Python, а также как считывать данные из таблицы и затем их использовать в Unity.
Также я вспомнил как создавать диаграммы в Google таблице и вспомнил синтаксис Python.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**
