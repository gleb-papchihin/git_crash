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

Если планируете импортировать или создавать свои библиотеки, то имейте в виду, что blender использует свое виртуальное окружение. Доступ к python-библиотекам можно получить по следующему пути: path_to_blender/3.0/python/lib/python3.9


## Объекты и коллекции
  
Список объектов и коллекций, с которыми мы можем работать, представлен в этом блоке:
  
