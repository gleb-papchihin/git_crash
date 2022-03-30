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


### scale

Масштабировать объект можно не только с помощью свойства dimensions, но и scale. В отличии от dimensions, scale не использует единицы измерения, а меняет масштаб в относительно исходного размера.

``` python
print(cube.scale)
# <Vector (1.0000, 1.0000, 1.0000)>

cube.scale = Vector((2, 1, 1)) # Используется mathutils.Vector, встроенный в blender

# Эквиалентно

cube.scale.x = 2
```

![location](https://github.com/gleb-papchihin/git_crash/blob/master/scale.gif)


### rotation_euler

Для вращения объекта используется свойство rotation_euler. Оно немного отличается от location, dimensions и scale.

``` python
import math

print(cube.rotation_euler)
# <Euler (x=0.0000, y=0.0000, z=0.0000), order='XYZ'>

cube.rotation_euler = (
    math.radians(60),
    math.radians(45),
    math.radians(30)
    )

# Эквивалентно

cube.rotation_euler.x = math.radians(60)
cube.rotation_euler.y = math.radians(45)
cube.rotation_euler.z = math.radians(30)

print(cube.rotation_euler)
# <Euler (x=1.0472, y=0.7854, z=0.5236), order='XYZ'>
```

Предыдущий подход перезаписывает углы. Но если мы хотим повернуть объект относительно текущего положения, то для этого можно использовать следующий метод.

``` python
rotation = Euler((0, 0, math.radians(10))) # Используется mathutils.Euler, встроенный в blender
cube.rotation_euler.rotate(rotation)
```

При вращении объекта с помощью углов Эйлера, нужно помнить об одной из их особенностей — gimble lock. Про это можно прочитать [тут](https://habr.com/ru/post/183116/).

![rotation](https://github.com/gleb-papchihin/git_crash/blob/master/rotation.gif)

### bound_box

Это свойство доступно только для чтения. Оно возвращает 8 точек, описывающих границы объекта. В отличии от location, dimensions и scale, bound_box не использует mathutils.Vector.

``` python
print(type(cube.bound_box))
# <class 'bpy_prop_array'>

print(type(cube.bound_box[0]))
# <class 'bpy_prop_array'>
```

Для удобства преобразуем точки в mathutils.Vector.

``` python
def bounding_box(obj):
    return [Vector(point) for point in obj.bound_box]
```

У этого свойства есть еще одна небольшая особенность: по умолчанию координаты граничных точек рассчитываются относительно центра объекта, а не сцены. Чтобы перевести координаты точек в систему сцены, можно использовать следующую функцию.

``` python
def bounding_box_world(obj):
    return [obj.matrix_world @ point for point in bounding_box(obj)]
```

![bbox](https://github.com/gleb-papchihin/git_crash/blob/master/bbox.gif)
