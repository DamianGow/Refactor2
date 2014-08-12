replace-exception-with-test:php

###

1. Создайте условный оператор для граничного случая и поместите его перед <code>try</code>/<code>catch</code> блоком.

2. Переместите код из <code>catch</code>-секции внутрь этого условного оператора.

3. В <code>catch</code>-секции поставьте код выбрасывания обычного безымянного исключения и запустите все тесты.

4. Если никаких исключений не было выброшено во время тестов, избавьтесь от оператора <code>try</code>/<code>catch</code>.



###

```
class ResourcePool {
  // ...
  private $available; // SplStack 
  private $allocated; // SplStack 

  public function getResource() {
    try {
      $result = $this->available->pop();
      $this->available->push($this->allocated, $result);
      return $result;
    } catch (RuntimeException $e) {
      $result = new Resource();
      $this->available->push($this->allocated, $result);
      return $result;
    }
  }
}
```

###

```
class ResourcePool {
  // ...
  private $available; // SplStack 
  private $allocated; // SplStack 

  public function getResource() {
    if ($this->available->isEmpty()) {
      $result = new Resource();
    }
    else {
      $result = $this->available->pop();
    }
    $this->available->push($this->allocated, $result);
    return $result;
  }
}
```

###

Set step 1

# Для этого примера возьмём объект, управляющий ресурсами, создание которых обходится дорого, но возможно повторное их использование. Хороший пример такой ситуации дают соединения с базами данных.

Select "private |||$available|||"
#+ У администратора соединений есть два пула, в одном из которых находятся ресурсы, доступные для использования

Select "private |||$allocated|||"

#<= а в другом – уже выделенные.

Select "$this->available->pop()"

#< Когда клиенту нужен ресурс, администратор предоставляет его из пула доступных и переводит в пул выделенных. Когда клиент высвобождает ресурс, администратор возвращает его обратно.

Select "$result = new Resource();"

#< Если клиент запрашивает ресурс, когда свободных ресурсов нет, администратор создаёт новый ресурс.

#< В данном случае нехватка ресурсов не является неожиданным происшествием, поэтому использование исключения не совсем оправдано.

Go to the start of "getResource"

# Итак, попытаемся избавиться от исключения. Первым делом в начале метода создадим условный оператор с условием совпадающим с условием выброса исключения. Весь остальной код поместим в <code>else</code>.

Print:
```

    if ($this->available->isEmpty()) {
    }
    else {
```

Go to:
```
    }|||
  }
```

Print:
```

    }
```

Select:
```
    try {
      $result = $this->available->pop();
      $this->available->push($this->allocated, $result);
      return $result;
    } catch (RuntimeException $e) {
      $result = new Resource();
      $this->available->push($this->allocated, $result);
      return $result;
    }

```

Indent

Set step 2

Select:
```
        $result = new Resource();
        $this->available->push($this->allocated, $result);
        return $result;

```

# Теперь, скопируем код из <code>catch</code> секции внутрь граничного условного оператора.

Go to "isEmpty()) {|||"

Print:
```

      $result = new Resource();
      $this->available->push($this->allocated, $result);
      return $result;
```

Set step 3

Go to "catch (RuntimeException $e) {|||"

# Теперь код никогда не должен достигать <code>catch</code> секции. Но чтобы убедиться в этом на 100%, вставим проверку внутрь секции и запустим все тесты.

Print:
```

        throw new RuntimeException("Should not reach here.");
```

#C Посмотрим, что покажет компиляция и авто-тесты.

#S Всё отлично, можем продолжать!

Set step 4

# Теперь, мы можем удалить try/catch секцию, не беспокоясь о возможных ошибках.

Select:
```
      try {

```

Remove selected

Select:
```
      } catch (RuntimeException $e) {
        throw new RuntimeException("Should not reach here.");
        $result = new Resource();
        $this->available->push($this->allocated, $result);
        return $result;
      }

```

Remove selected

Select:
```
        $result = $this->available->pop();
        $this->available->push($this->allocated, $result);
        return $result;
```

Deindent

Select:
```
      $this->available->push($this->allocated, $result);
      return $result;

```

# Обычно после этого оказывается возможным привести в порядок условный код. В данном случае мы можем применить <a href="/consolidate-duplicate-conditional-fragments">консолидацию дублирующихся условных фрагментов</a>.

Go to:
```
    }|||
  }
```

Print:
```

    $this->available->push($this->allocated, $result);
    return $result;
```

Wait 500ms

Select:
```
      $this->available->push($this->allocated, $result);
      return $result;

```

Remove selected

#C Запускаем финальную компиляцию.

#S Отлично, все работает!

Set final step

#Q На этом рефакторинг можно считать оконченным. В завершение, можете посмотреть разницу между старым и новым кодом.