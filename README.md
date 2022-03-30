# Собираем генератор данных на Blender. Часть 1: Объекты

Привет, Хабр! Меня зовут Глеб. Я работаю в компании Friflex над проектом idChess (приложением для распознавания и аналитики шахматных партий). Во время работы у нас появилась потребность в улучшении качества распознавания, поэтому мы решили расширить наш датасет синтетическими данными. В качестве движка выбрали blender (у него приятный API, написанный на python). Когда дело дошло до реализации генератора, пришлось покопаться в документации и потыкаться в консоли. Чтобы помочь тем, кто решил собрать генератор данных для своего проекта, но еще не имеет опыта, хочу рассказать про базовые возможности интерфейса и наблюдения, которые сделал во время работы над задачей. 

**Содержание:**

- Часть 1: Объекты. В этом блоке научимся взаимодействовать с объектами. Разберем получение доступа через API, перемещение, масштабирование и вращение
- Часть 2: Камера. Разберемся, как автоматически настроить фокусное расстояние, навести камеру на предмет и отобразить координаты объекта на кадр
- Часть 3: Материалы и освещение. Рассмотрим свойства источников света и работу с материалами через пользовательские свойства и драйверы
- Часть 4: Сборка проекта и рендеринг. Научимся загружать объекты из разных файлов и запускать рендеринг сцены через консоль


## Интерфейс

Чтобы начать работу с API блендера, откроем python-консоль внутри приложения. Через нее мы будем запускать наши скрипты и наблюдать изменения на сцене.

![интерфейс](https://github.com/gleb-papchihin/git_crash/blob/master/console.gif)

Помимо консоли, в blender есть текстовый редактор, который позволяет запускать код прямо в приложении. Более подробно про редактор можно посмотреть [тут](https://youtu.be/yg5CWcj-BM4?list=PLOVSu7-KesPgPiatDTP7jvdgxxwp18LyH&t=129). 

Если планируете импортировать или создавать свои библиотеки, то имейте в виду, что blender использует свое виртуальное окружение. Доступ к python-библиотекам можно получить по следующему пути: /blender/3.0/python/lib/python3.9


## Объекты и коллекции
  
Список объектов и коллекций, с которыми мы можем работать, представлен в этом блоке:
  
![блок с объектами и коллекциями](https://github.com/gleb-papchihin/git_crash/blob/master/objects.png)

Коллекции по своей сути похожи на обычные папки за одним исключением: при попытке получить содержимое коллекции мы получим **все** объекты — даже те, которые содержатся в дочерних коллекциях. Например:

``` python
bpy.data.collections['Collection'].all_objects.values()

# [bpy.data.objects['Cube'], bpy.data.objects['Light'], bpy.data.objects['Camera']]
```

Все объекты в списке из примера выше наследуются от класса bpy.types.Object, который предоставляет доступ к свойствам (положение, масштаб, поворот и т.д.). Но перед тем, как перейти к рассмотрению методов, разберемся, как получить доступ к экземплярам.

``` python
cube = bpy.data.objects['Cube']
```

bpy.data.objects ведет себя, как словарь: доступны методы values, keys, items, get и т.д. А ключами являются названия объектов. Хорошо, а что делать, если мы хотим получить объекты, содержащиеся в какой-нибудь коллекции? Просто используем другой атрибут модуля bpy.data:

``` python
bpy.data.collections['Collection'].all_objects.values()

# [bpy.data.objects['Cube'], bpy.data.objects['Light'], bpy.data.objects['Camera']]
```

Объекты получены. Теперь пройдемся по их свойствам.

**Важно:** свойства, которые мы рассмотрим далее, являются дескрипторами. А что это значит для нас? Например, нам не нужно искать метод «move», чтобы переместить объект — достаточно изменить location — свойство отвечающее за положение.

### location

По умолчанию это свойство отвечает за смещение относительно центра сцены. Попробуем переместить объект «Cube».

``` python
print(cube.location)
# <Vector (0.0000, 0.0000, 0.0000)>

cube.location = (1, 0, 2)

# Эквивалентно

cube.location.xz = (1, 2)

# Эквивалентно

cube.location.x = 1
cube.location.z = 2

# Эквивалентно

cube.location[0] = 1
cube.location[2] = 2

print(cube.location)
# <Vector (1.0000, 0.0000, 2.0000)>
```

Для болшей наглядности, рассмотрим более сложный пример.

``` python
from threading import Thread
import time
import math

def apply_simple_animation(cube, period):
    for deg in range(361):
        cube.location.x = 5 * math.sin(math.radians(deg))
        time.sleep(period)

Thread(target = apply_simple_animation, args = (cube, 0.01)).start()
```

![location](https://github.com/gleb-papchihin/git_crash/blob/master/location.gif)

### dimensions

Это свойство позволяет растягивать и сжимать объект вдоль осей. Dimensions возвращает значения в единицах измерения (по умолчанию это метры).

``` python
print(cube.dimensions)
# <Vector (2.0000, 2.0000, 2.0000)>

cube.dimensions = (4, 2, 2)

# Эквивалентно

cube.dimensions.x = 4

# Эквивалентно

cube.dimensions[0] = 4

print(cube.dimensions)
# <Vector (4.0000, 2.0000, 2.0000)>
```

Однако заметим, что для некоторых объектов dimension работать не будет. Об этом мы поговорим далее, а пока посмотрим на простой пример.

``` python
camera = bpy.data.objects['Camera']

print(camera.dimensions)
# <Vector (0.0000, 0.0000, 0.0000)>

camera.dimensions.x = 2

print(camera.dimensions)
# <Vector (0.0000, 0.0000, 0.0000)>
```

![location](https://github.com/gleb-papchihin/git_crash/blob/master/dimensions.gif)
