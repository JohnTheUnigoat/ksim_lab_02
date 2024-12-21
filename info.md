## Комп'ютерні системи імітаційного моделювання
## СПм-23-4, **Гудзинський Іван Вікторович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 6, модель у середовищі NetLogo:
[Rabbits Grass Weeds](http://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Biology/Rabbits%20Grass%20Weeds.nlogo). Класична модель "заячого лугу".

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:
Додати можливість отруїтися при поїданні бур'янів (зазначена у внутрішніх параметрах, як певна вірогідність).

Процедура eat-weeds була змінена, тепер має такий вигляд:
```
to eat-weeds ;; rabbit procedure
  ;; gain "weed-energy" by eating weeds and have a chance of getting sick
  if pcolor = violet [
    set pcolor black
    set energy energy + weed-energy

    let sickness-roll (random 100) + 1 ; 1-100 roll
    if sickness-roll <= weeds-sickness-chance [
      get-sick
    ]
  ]
end
```

Захворілий кролик не може харчуватися, переміщатися і розмножуватися:
```
to go
  if not any? rabbits [ stop ]

  grow-grass-and-weeds
  ask rabbits [
    ifelse is-sick [
      be-sick
    ][
      move
      eat-grass
      eat-weeds
      reproduce
    ]
    die-if-tired
  ]
  tick
end
```

Захворілий кролик позначається іншим кольором і залишається хворим на 3 такти модельного часу:

```
to get-sick ;; rabbit procedure
  ; get sick for 3 ticks
  set is-sick true
  set sick-ticks-remaining 3
  set color gray
end

to get-healthy ;; rabbit procedure
  set is-sick false
  set-color-by-sex
end

to be-sick ;; rabbit procedure
  set sick-ticks-remaining sick-ticks-remaining - 1
  set energy energy - sick-tick-energy-cost

  if sick-ticks-remaining = 0 [
    set is-sick false
    set-color-by-sex
  ]
end
```

Додати поділ кроликів на самців та самок:

```
rabbits-own [
  energy
  sex
  is-sick
  sick-ticks-remaining
]

to init-rabbit [init-x init-y] ;; rabbit procedure
  set sex one-of ["male" "female"]
  set-color-by-sex
  setxy init-x init-y
  set energy random 10
  set is-sick false
  set sick-ticks-remaining 0
end

to set-color-by-sex ;; rabbit procedure
  set color (ifelse-value
      sex = "male" [sky]
      sex = "female" [pink]
    )
end
```

Поява нових кроликів має вимагати не тільки ситості, а й здоров'я, та присутності в одній із сусідніх клітин іншого ситого здорового кролика протилежної статі. Поява потомства відбувається із ймовірністю 50%

```
to reproduce ;; rabbit procedure
  if energy > birth-threshold [
    let partner-sex other-sex sex

    let possible-partners rabbits in-radius 1 with [
      sex = partner-sex and
      energy > birth-threshold and
      not is-sick
    ]

    if any? possible-partners [
      let partner min-one-of possible-partners [distance myself]
      ask partner [
        set energy energy / 2
      ]
      set energy energy / 2

      let is-reproduction-successfull one-of [true false]

      if is-reproduction-successfull [
        hatch 1 [
          init-rabbit [xcor] of myself [ycor] of myself
          fd 1
        ]
      ]
    ]
  ]
end

to-report other-sex [my-sex]
  report (ifelse-value
    sex = "male" ["female"]
    sex = "female" ["male"]
  )
end
```

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

Самці та самки розділяються за кольором (коли не хворіють):

```
to set-color-by-sex ;; rabbit procedure
  set color (ifelse-value
      sex = "male" [sky]
      sex = "female" [pink]
    )
end
```

Експеримент завершується не лише коли зайців більше не залишилося, а ще й коли залишились лише зайці одної статі:

```
to go
  ; stop if there are no more rabbits or if all remaining rabbits are of the same sex
  if not any? rabbits [ stop ]
  if not any? rabbits with [sex = "male"] [ stop ]
  if not any? rabbits with [sex = "female"] [ stop ]
  ...
```

Під час хвороби зайці не лише не можуть нічого робити, а ще й витрачають певну кількість енергії коже тік. Ця кількість є зовнішнім параметром `sick-tick-energy-cost`

```
to be-sick ;; rabbit procedure
  set sick-ticks-remaining sick-ticks-remaining - 1
  set energy energy - sick-tick-energy-cost
  
  if sick-ticks-remaining = 0 [
    set is-sick false
    set-color-by-sex
  ]
end
```

Фінальний код моделі доступний за [посиланням](rabbits_grass_weed.nlogo)
За цим [посиланням](https://github.com/JohnTheUnigoat/ksim_lab_02/compare/initial_model...updated_model) можна побачити всі внесені зміни в вихідну модель.

## Обчислювальні експерименти

### 1. Вплив витрат енергії під час хвороби на середнє значення популяції

Досліджується вплив витрат енергії в тіки під час яких заяць хворіє на середнє значення популяції зайців. Експерименти проводяться на діапазоні витрат енергії за тік від 0 до 4 з кроком 0.5, всього 9 симуляцій. Всі інші параметри мають наступні значення:
- **initial-rabbit-count**: 100
- **birth-threshold**: 15
- **grass-grow-rate**: 10
- **grass-energy**: 5
- **weeds-grow-rate**: 5
- **weeds-energy**: 2
- **weeds-sickness-chance**: 50

Проводиться спостереження за наступними показниками симуляції:
- Середня кількість зайців

Під "середньою популяцією" зайців мається на увазі середнє значення популяції зайців на встьому проміжку експерименту окрім початку (не беруться до уваги перші 400 тіків, бо в цей проміжок існує сильний вплив початкової популяції на коливання середнього значення).

| № Експерименту | Енергетичні трати на тік хвороби | Середня популяція |
|----------------|----------------------------------|-------------------|
| 1              | 0                                | 130               |
| 2              | 0.5                              | 120               |
| 3              | 1                                | 107               |
| 4              | 1.5                              | 89                |
| 5              | 2                                | 75                |
| 6              | 2.5                              | 61                |
| 7              | 3                                | 44                |
| 8              | 3.5                              | 31                |
| 9              | 4                                | 18                |

![image](https://github.com/user-attachments/assets/213d73b5-a188-4b48-a1ec-3694dcfe1f02)

Наглядно видно, що "суворість" захворювання радикально впливає на можливу популяцію при всіх інших рівних умовах.
