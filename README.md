## Комп'ютерні системи імітаційного моделювання
## СПм-24-4, **Почуєнков Владислав Ігорович**
### Лабораторна робота №**2**. Редагування імітаційних моделей у середовищі NetLogo

<br>

### Варіант 13, модель у середовищі NetLogo:
[Fruit Wars](https://www.netlogoweb.org/launch#http://www.netlogoweb.org/assets/modelslib/Sample%20Models/Social%20Science/Economics/Fruit%20Wars.nlogo)

<br>

### Внесені зміни у вихідну логіку моделі, за варіантом:

**1. Додаємо в інтерфейс моделі новий повзунок poison-chance** який буде контролювати появу отруйних кущів.  
![Додаємо в інтерфейс новий повзунок](slider.png)

**2.Глобальні змінні (globals).** Додали лічильники для статистики смертей від отрути. (Примітка: poison-chance тут немає, бо він створений як повзунок в інтерфейсі): 
<pre>
globals [
  ; ... старі змінні ...
  poison-death-count ; Лічильник смертей від отрути
  poison-death-rate  ; Відсоток смертей від отрути
]
</pre>

**3. Властивості кущів (fruit-bushes-own).** Додали "прапорець", який визначає, чи кущ отруйний:
<pre>
 fruit-bushes-own [
  amount
  poisonous? ; true = отруйний, false = їстівний
]
</pre>

**3. Властивості збирачів (foragers-own).** Додали таймер хвороби та пам'ять про початкову швидкість:
<pre>
foragers-own [
  ; ... старі змінні ...
  poisoned-ticks-remaining ; Скільки тактів залишилося бути "хворим"
  original-speed           ; Щоб повернути швидкість після одужання
]
</pre>

**4. Генерація отрути.** Змінюємо процедуру створення кущів (to grow-fruit-bushes), щоб деякі з них ставали отруйними та змінювали колір фруктів на чорний:
<pre>
sprout-fruit-bushes 1 [
  set shape "Bush"
  
  ; --- Нова логіка ---
  if random 100 < poison-chance [ ; Використовуємо значення з повзунка
    set poisonous? true
    set color black ; Отруйний = чорний
  ] [
    set poisonous? false
    set color one-of [ red blue orange yellow ] ; Звичайний
  ]
  ; ...
]
</pre>



<br>

### Внесені зміни у вихідну логіку моделі, на власний розсуд:

**Додано ймовірність безвідповідальності водія, який, при нагоді, їздитиме узбіччям**.
Імовірність встановлюється користувачем через інтерфейс середовища моделювання (слайдер для bad-driver-probability) та використовується при додаванні машин на полі:
<pre>
globals [
  sample-car
  speed-min
  car-roadside-amount
  bad-driver-probability
]

;; виїзд на узбіччя здійснюється, якщо перед машиною є інша машина і ще одна, водій "поганий", знаходиться на дорозі (бо з узбіччя з'їжджати далі нікуди)
if (car-ahead-1 != nobody) [
    ;; перевіряємо, чи буде на даному тіку водій "недисциплінованим"
    let rand random 100
    ifelse(rand < bad-driver-probability) [
      ;; якщо недисциплінований, і при цьому є додаткова перешкода перед машинойї перед нами, то переміщуємось на узбіччя
      if(car-ahead-2 != nobody) [
        ;; перевірка, чи знаходимось ми зараз на дорозі і чи вільне узбіччя
        if (is-on-road) and (roadside-free-check) [
          set heading 180
          fd 1
          set heading 90
          ;; збільшуємо лічильник "узбічників"
          set car-roadside-amount car-roadside-amount + 1
        ]
      ]
    ]
  ]
</pre>
Лічильник "поганих водіїв" виводитиметься користувачеві.

Перед виїздом на узбіччя, воне проглядається водієм, чи вільно там, за допомогою функції is-roadside-free:
<pre>
to-report roadside-free-check
  set heading 180
  let car-roadside one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-roadside = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

Пеервірка, чи знаходимось на дорозі:
<pre>
to-report is-on-road
    report pycor = 0
end
</pre>

Додано узбіччя, дорога зведена до повноційної односмугової:
<pre>
to setup-road ;; patch procedure
  if pycor < 1 and pycor > -2 [ set pcolor yellow ]
  if pycor < 1 and pycor > -1 [ set pcolor white ]
end
</pre>

На кожному ході виконується перевірка, чи знаходиться машина на дорозі. Якщо машина їде узбіччям, то намагатиметься повернутися на дорогу:
<pre>
  if(not is-on-road) [
      if (road-main-free-check) [
        set heading 0
        fd 1
        set heading 90
        set car-roadside-amount car-roadside-amount - 1
      ]
    ]
</pre>
Функція перевірки, чи вільна дорога поблизу машини, щоб можна було повернутися з узбіччя:
<pre>
to-report road-main-free-check
  set heading 0
  let car-road one-of turtles-on patch-ahead 1
  set heading 90
  ;ifelse(car-roadside = nobody) [
    report car-road = nobody
  ;;]
  ;;[
  ;;  report false
  ;;]
end
</pre>

![Скріншот моделі в процесі симуляції](example-model.png)

Фінальний код моделі та її інтерфейс доступні за [посиланням](example-model.nlogo). *// якщо вносили зміни до інтерфейсу середовища моделювання - то експорт потрібен у форматі nlogo, як тут. Інакше, якщо змінювався лише код логіки моделі, достатньо викласти лише його, як [тут](example-model-code.html),якщо експортовано з десктопної версії NetLogo, або окремим текстовим файлом, шляхом копіпасту з веб-версії*.
<br>

## Обчислювальні експерименти
*// тут повинен бути наведений опис одного експерименту, за аналогією з першої л/р.* 
### 1. Вплив дисциплінованості водіів на середню швидкість переміщення
