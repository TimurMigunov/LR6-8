ЛР6-8
Мигунов Т.У.
ЭВТ-70

Игровой Движок: Unity.

####***Лабараторная работа №6-8***

Тема: разработка игры симулятора фермы

Цель: приобрести навыки в разработке симулятора фермы

1.	Выполнение работы

Импортирование ресурсов игры

_______________![1](https://user-images.githubusercontent.com/119228138/204732133-76f428be-6794-4ff6-8457-7c5fcca10084.png)

_______________Рис. 6.1 Импорт ресурсов

![1](https://user-images.githubusercontent.com/119228138/204732362-6e9c6c72-ba03-4f68-a27b-b64f3e764829.png)

_______________Рис. 6.2 – Игровой объект

3.	Иерархия объектов

_______________![image](https://user-images.githubusercontent.com/119228138/204732489-63848926-5ec7-4141-b14a-a1c55b0a1d1c.png)

_______________Рис. 6.3 - Иерархия объектов

4.	Скрипты игрового объекта

_______________![1](https://user-images.githubusercontent.com/119228138/204753363-b6c71c2d-639e-4593-a53c-3870840a2fe0.png)

____________Рис. 6.4 - Скрипты игрового объект

```
5.	Скрипт FarmManager
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;
using UnityEngine.UI;
public class FarmManager : MonoBehaviour
{
    public PlantItem selectedPlant;
    public bool isPlanting = false;
    public int money = 0;
    public TextMeshProUGUI moneyTxt;
    public bool isSelecting = false;
    //1-water 2-Fertilizer 3-buy_cell
    public int selectedTool = 0;
    public Image[] buttonsImg;
    public Sprite normalButton;
    public Sprite selectedButton;
    public Color buyColor = Color.green;
    public Color cancelColor = Color.red;
    void Start() =>  moneyTxt.text = ("$" + money);
    public void SelectedPlant(PlantItem _newPlant)
    {
        if (selectedPlant == _newPlant)
        {
            CheckSelectionTool();
        }
        else
        {
            CheckSelectionTool();
            selectedPlant = _newPlant;
            selectedPlant.btnImage.color = cancelColor;
            selectedPlant.btnText.text = "Cancle";
            isPlanting = true;
        }
    }
    public void SelectTool(int toolNumber)
    {
        if (toolNumber == selectedTool)
        {
            //deselect
            CheckSelectionTool();
        }
        else
        {
            //selected
            CheckSelectionTool();
            isSelecting = true;
            selectedTool = toolNumber;
            buttonsImg[toolNumber - 1].sprite = selectedButton;
        }
    }
    private void CheckSelectionTool()
    {
        if (isPlanting)
        {
            isPlanting = false;
            if (selectedPlant != null)
            {
                selectedPlant.btnImage.color = buyColor;
                selectedPlant.btnText.text = "Buy";
                selectedPlant=null;
            }
        }
        if (isSelecting)
        {
            if (selectedTool>0)
            {
                buttonsImg[selectedTool - 1].sprite = normalButton;
            }
            isSelecting= false;
            selectedTool = 0;
        }
    }
    public void Transaction(int _value)
    {
        money += _value;
        moneyTxt.text = ("$" + money);
    }
}
6.	Скрипт IzometricZ
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
public class IzometricZ : MonoBehaviour
{
    void Start()
    {
        transform.position = new Vector3(transform.position.x, transform.position.y, transform.position.y / 100);
    }
}
7.	Скрипт PlantItem

using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;
using UnityEngine.UI;
public class PlantItem : MonoBehaviour
{
    public PlantObject plant;
    public TextMeshProUGUI nameTxt;
    public TextMeshProUGUI priceTxt;
    public Image icon;
    public Image btnImage;
    public TextMeshProUGUI btnText;
    private FarmManager fm;
    void Start()
    {
        fm = FindObjectOfType<FarmManager>();

        InititializeUI();
    }

    void Update()
    {
        
    }

    public void BuyPlant()
    {
        Debug.Log($"Bought {plant.plantName}");
        fm.SelectedPlant(this);
    }
    void InititializeUI()
    {
        nameTxt.text = plant.plantName;
        priceTxt.text = "$" + plant.buyPrice;
        icon.sprite = plant.icon;
    }
}


8.	Скрипт PlantObject
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
[CreateAssetMenu(fileName ="New Plant", menuName ="Plant")]
public class PlantObject : ScriptableObject
{
    public string plantName;
    public Sprite[] plantStages;
    public float timeBtwStages;
    public int buyPrice;
    public int sellPrice;
    public Sprite icon;
    public Sprite dryPlanted;
}
9.	Скрипт PlotManager
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;
public class PlotManager : MonoBehaviour
{
    public Color availableColor = Color.green;
    public Color unavailableColor = Color.red;
    public Sprite drySpritePlot;
    public Sprite normalSpritePlot;
    public Sprite unavailableSprite;
    public bool isBought = true;
    private SpriteRenderer plot;
    private SpriteRenderer plant;
    private BoxCollider2D plantCollider;
   
    private bool isPlanted = false;
    private int stage = 0;
    private float timer;
    private PlantObject selectedPlant;
    private FarmManager fm;
    private bool isDry = true;
    private float speed =1f;
    void Start()
    {
        plant = transform.GetChild(0).GetComponent<SpriteRenderer>();
        plantCollider = transform.GetChild(0).GetComponent<BoxCollider2D>();
        fm = transform.GetComponentInParent<FarmManager>();
        plot = GetComponent<SpriteRenderer>();
        if (isBought)
        {
            plot.sprite = drySpritePlot;
        }
        else
        {
            plot.sprite = unavailableSprite;
        }
    }
    void Update()
    {
        if (isPlanted && !isDry)
        {
            timer -= speed*Time.deltaTime;
            if (timer < 0 && stage < selectedPlant.plantStages.Length-1)
            {
                timer = selectedPlant.timeBtwStages;
                stage++;
                UpdatePlant();
            }
        }
    }

    private void OnMouseDown()
    {
        if (isPlanted)
        {
            if(stage == selectedPlant.plantStages.Length-1 && !fm.isPlanting && !fm.isSelecting)
                Harvest();
        }
        else if(fm.isPlanting && fm.selectedPlant.plant.buyPrice <=fm.money && isBought)
        {
            Plant(fm.selectedPlant.plant);
        }
        if (fm.isSelecting)
        {
            switch (fm.selectedTool)
            {
                case 1:
                    if (fm.money >= 5 && isBought && plot.sprite != normalSpritePlot)
                    {
                        fm.Transaction(-5);
                        isDry = false;
                        plot.sprite = normalSpritePlot;
                        if(isPlanted) 
                            UpdatePlant();
                    }
                    break;
                case 2:
                    if (fm.money>=70 && isBought)
                    {
                        if (speed < 2)
                        {
                            fm.Transaction(-70);
                            speed += .2f;
                        }

                    }
                    break;
                case 3:
                    if (fm.money >= 500 && !isBought)
                    {
                        fm.Transaction(-500);
                        isBought = true;
                        plot.sprite = drySpritePlot;
                    }
                    break;
                default:
                    break;
            }
        }
    }
    private void OnMouseOver()
    {
        if (fm.isPlanting)
        {
            if (isPlanted || fm.selectedPlant.plant.buyPrice > fm.money || !isBought)
            {
                plot.color = unavailableColor;
            }
            else
            {
                plot.color = availableColor;
            }
        }
        if (fm.isSelecting)
        {
            switch (fm.selectedTool)
            {
                case 1:
                case 2:
                    if (isBought && fm.money>=(fm.selectedTool-1)*70)
                        plot.color = availableColor;

                    else
                        plot.color = unavailableColor;
                    break;
                case 3:
                    if (!isBought && fm.money>=500)
                        plot.color = availableColor;

                    else
                        plot.color = unavailableColor;
                    break;
                default:
                    plot.color = unavailableColor;
                    break;
            }
        }
    }

    private void OnMouseExit() => plot.color = Color.white;
    private void Harvest()
    {  
        isPlanted = false;
        plant.gameObject.SetActive(false);
        fm.Transaction(selectedPlant.sellPrice);
        isDry = true;
        plot.sprite = drySpritePlot;
        speed = 1f;
    }
    private void Plant(PlantObject newPlant)
    {
        selectedPlant = newPlant;
        isPlanted = true;
        fm.Transaction(-selectedPlant.buyPrice);
        stage = 0;
        UpdatePlant();
        timer = selectedPlant.timeBtwStages;
        plant.gameObject.SetActive(true);
    }
    private void UpdatePlant()
    {
        if (isDry)
        {
            plant.sprite = selectedPlant.dryPlanted;
        }
        else
        {
        plant.sprite = selectedPlant.plantStages[stage];
        }
        plantCollider.size = plant.sprite.bounds.size;
        plantCollider.offset = new Vector2(0, plant.bounds.size.y/2);
    }
}

10.	Скрипт StoreManager
using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices.WindowsRuntime;
using UnityEngine;
public class StoreManager : MonoBehaviour
{
    public GameObject plantItem;
    List<PlantObject> plantObj = new List<PlantObject>();
    private void Awake()
    {
        var loadPlants = Resources.LoadAll("Plants", typeof(PlantObject));
        foreach (var _plant in loadPlants)
        {
            plantObj.Add((PlantObject)_plant);
        }
        plantObj.Sort(SortByPrice);
        foreach (var _plant in plantObj)
        {
            PlantItem newPlant = Instantiate(plantItem, transform).GetComponent<PlantItem>();
            newPlant.plant = _plant;
        }
    }
    private int SortByPrice(PlantObject _plantObject1, PlantObject _plantObject2) => _plantObject1.buyPrice.CompareTo(_plantObject2.buyPrice);
    private int SortByTime(PlantObject _plantObject1, PlantObject _plantObject2) => _plantObject1.timeBtwStages.CompareTo(_plantObject2.timeBtwStages);
}
```

11.	Настройка объектов Grid и Farm

__________________![1](https://user-images.githubusercontent.com/119228138/204764032-b763e617-f05d-4b22-92f9-4b67712474cd.png)

__________________Рис. 6.5. Процесс настройки объекта Grid

__________________![1](https://user-images.githubusercontent.com/119228138/204764583-340986a7-1eb9-4c2d-bd9c-51ec371a2845.png)

__________________Рис. 6.6 Процесс добавления объекта Farm

__________________ ![1](https://user-images.githubusercontent.com/119228138/204764973-b8370c23-795c-4a3e-83a2-62677c548500.png)

__________________ Рис. 6.7 Настройка объекта Farm

__________________ ![1](https://user-images.githubusercontent.com/119228138/204765272-290f87e3-8048-4495-8fc0-16b4867fa0ad.png)

__________________Рис. 6.8 Настройка объекта Ground

Вывод: в проделанная лабораторной работе – был создан симулятор фермы.

[LR6-8.docx](https://github.com/TimurMigunov/Lab./files/10115794/LR6-8.docx)

[lr6-8.ZIP](https://github.com/TimurMigunov/LR6-8/files/10116618/lr6-8.ZIP)
