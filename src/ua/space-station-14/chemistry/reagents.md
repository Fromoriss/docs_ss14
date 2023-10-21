# Реагенти

Реагенти це речовини, що входять до складу розчину і здатні вступати в реакції з утворенням нових реагентів.

Визначення реагенту складається з:
"тип": Назва компонента. Завжди має бути `reagent`.
`id`: Унікальний ідентифікатор реагенту. Використовується в реакціях та інших місцях для ідентифікації реагентів. Це може бути `Iodine` або `WeldingFuel`. Формат запису - `PasacalCase`.
`name`: Ідентифікатор записаний звичайним текстом, для відображення у підказках. Наприклад, `iodine` і `welding fuel`. Ім'я не обов'язково має бути таким самим, що і `id`, що дозволяє перекладати назви реагентів без наслідків.
`parent`: Від якого реагенту успадковувати властивості. Використовується для таких речей, як BaseDrink.
`desc`: Читабельний для людини опис речовини.
`color`: Колір реагенту, записаний в шістнадцятковій системі числення. Використовується для змішування та відображення хімічних речовин у контейнерах.
`spritePath`: Шлях до спрайту, починаючи з `Textures/Objects/Consumable/Drinks`. Він визначає піктограму, яка використовується для реагенту.
`physicalDesc`: Розмитий опис що потенційно може бути корисним при ідентифікації.
`boilingPoint`: Температура в градусах Цельсія, при якій реагент переходить з рідини в газ.
`meltingPoint`: Температура в градусах Цельсія, при якій реагент переходить з твердого стану в рідкий.
`metabolisms`: Словник типу MetabolismGroup->List\<ReagentEffect\>, з ефектами, що буде застосовано, коли буде реагент спожито.

## Приклад визначення реагенту

Так виглядає прототип реагенту в оригінальному коді:

```yaml
- type: reagent
  id: Lemonade
  name: lemonade
  parent: BaseDrink
  desc: Drink using lemon juice, water, and a sweetener such as cane sugar or honey.
  physicalDesc: tart
  color: "#FFFF00"
  spritePath: lemonadeglass.rsi
  metabolisms:
    Drink:
      effects:
      - !type:SatiateThirst
        factor: 2
```

А так виглядає той самий, але перекладений прототип:

```yaml
- type: reagent
  id: Lemonade
  name: лимонад
  parent: BaseDrink
  desc: Напій з лимонного соку, води та підсолоджувача, наприклад, тростинного цукру або меду.
  physicalDesc: tart
  color: "#FFFF00"
  spritePath: lemonadeglass.rsi
  metabolisms:
    Drink:
      effects:
      - !type:SatiateThirst
        factor: 2
```

Для отримання додаткової інформації дивіться: [SS14 Content: Reagent folder](https://github.com/space-wizards/space-station-14/tree/master/Resources/Prototypes/Reagents) або [SS14 Content:ReagentPrototype.cs](https://github.com/space-wizards/space-station-14/blob/ca50a5f9934a399826306659f298f0098251e4eb/Content.Shared/Chemistry/Reagent/ReagentPrototype.cs)

## Вплив та Умови Реагентів

Ефекти `ReagentEffect` описані у C#. Наприклад, `SatiateThirst` та `HealthChange`. Це дозволяє без пролем повторно використовувати код для реагентів, які мають подібні ефекти. Багато сутностей використовують ефекти реагентів - метаболізм персонажів, метаболізм рослин, реакції сутностей на реагенти (ефекти на основі дотику/ін'єкції), тощо. У `ReagentEffect` також є поле `prob`, яке вказує на ймовірність того, що ефект станеться, від 0 до 1. Таке узагальнення зроблено для того, щоб речі, які потребують цих даних, могли їх використати.

До кожного ефекту у списку метаболізаторів додаються умови `ReagentEffectCondition`. За замовчуванням жодних умов не додається, тому ефекти завжди застосовуються. Однак ви можете легко додати власну поведінку, щоб визначити, коли ефект має відбутися. Наприклад, `ReagentThreshold` дозволяє дуже легко визначити поведінку при передозуванні. Ось як це реалізовано у метамфетаміна:

```yml
  metabolisms:
    Poison:
      effects:
      - !type:HealthChange
        damage:
          types:
            Poison: 2.5
      - !type:HealthChange
        conditions:
        - !type:ReagentThreshold
          min: 10
        damage:
          types:
            Poison: 4 # це додається до базової шкоди від метамфетаміну
    Narcotic:
      effects:
      - !type:MovespeedModifier
        walkSpeedModifier: 1.3
        sprintSpeedModifier: 1.3
```

Це означає, що поведінка при передозуванні HealthChange виникає лише тоді, коли у вашому організмі є щонайменше 10u речовини. Вона має дві різні групи: 'Отрута' та 'Наркотик' (у коді 'Poison' та 'Narcotic' відповідно). Обидві групи метаболізуються серцем.