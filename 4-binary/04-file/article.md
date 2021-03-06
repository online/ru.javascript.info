# File и FileReader

Объект [File](https://www.w3.org/TR/FileAPI/#dfn-file) наследуется от объекта `Blob` и возможностями по взаимодействию с файловой системой.

Есть два способа его получить.

Во-первых, есть конструктор, похожий на `Blob`:

```js
new File(fileParts, fileName, [options])
```

- **`fileParts`** -- массив значений Blob/BufferSource/строковых.
- **`fileName`** -- имя файла, строка.
- **`options`** -- необязательный объект со свойством:
    - **`lastModified`** -- дата последнего изменения в формате таймстамп (целое число).

Во-вторых, чаще всего мы получаем файл из `<input type="file">` или через перетаскивание с помощью мыши, или из других интерфейсов браузера. В этом случае файл получает эту информацию из ОС.

Так как `File` наследует от `Blob`, у объектов `File` есть те же свойства, плюс:
- `name` -- имя файла,
- `lastModified` -- таймстамп для даты последнего изменения.

В этом примере мы получаем объект `File` из `<input type="file">`:

```html run
<input type="file" onchange="showFile(this)">

<script>
function showFile(input) {
  let file = input.files[0];

  alert(`File name: ${file.name}`); // например, my.png
  alert(`Last modified: ${file.lastModified}`); // например, 1552830408824
}
</script>
```

```smart
Через `<input>` можно выбрать несколько файлов, поэтому `input.files` -- псевдомассив выбранных файлов. Здесь у нас только один файл, поэтому мы просто берём `input.files[0]`.
```

## FileReader

[FileReader](https://www.w3.org/TR/FileAPI/#dfn-filereader) объект, цель которого: читать данные из `Blob` (и, следовательно, `File` тоже).

Данные передаются при помощи событий, так как чтение с диска может занять время.

Конструктор:

```js
let reader = new FileReader(); // без аргументов
```
    
Основные методы:

- **`readAsArrayBuffer(blob)`** -- считать данные как `ArrayBuffer`
- **`readAsText(blob, [encoding])`** -- считать данные как строку (кодировка по умолчанию: `utf-8`)
- **`readAsDataURL(blob)`** -- считать данные как base64-кодированный URL.
- **`abort()`** -- отменить операцию.

Выбор метода для чтения зависит от того, какой формат мы предпочитаем, как мы хотим далее использовать данные.

- `readAsArrayBuffer` - для бинарных файлов, для низкоуровневой побайтовой работы с бинарными данными. Для высокоуровневых операций у `File` есть свои методы, унаследованные от `Blob`, например, `slice`, мы можем вызвать их напрямую.
- `readAsText` - для текстовых файлов, когда мы хотим получить строку.
- `readAsDataURL` -- когда мы хотим использовать данные в `src` для `img` или другого тега. Есть альтернатива - можно не читать файл, а вызвать `URL.createObjectURL(file)`, детали в главе <info:blob>.

В процессе чтения происходят следующие события:
- `loadstart` -- чтение начато.
- `progress` -- срабатывает во время чтения данных.
- `load` -- нет ошибок, чтение окончено.
- `abort` -- вызван `abort()`.
- `error` -- произошла ошибка.
- `loadend` -- чтение завершено (успешно или нет).

Когда чтение закончено, мы сможем получить доступ к его результату следующим образом:
- `reader.result` результат чтения (если оно успешно)
- `reader.error` объект ошибки (при неудаче).

Наиболее часто используемые события - это, конечно же, `load` и `error`.

Вот пример чтения файла:

```html run
<input type="file" onchange="readFile(this)">

<script>
function readFile(input) {
  let file = input.files[0];

  let reader = new FileReader();

  reader.readAsText(file);

  reader.onload = function() {
    console.log(reader.result);
  };

  reader.onerror = function() {
    console.log(reader.error);
  };

}
</script>
```

```smart header="`FileReader` для Blob"
Как упоминалось в главе <info:blob>, `FileReader` работает для любых объектов Blob, а не только для файлов.

Поэтому мы можем использовать его для преобразования Blob в другой формат:
- `readAsArrayBuffer(blob)` -- в `ArrayBuffer`,
- `readAsText(blob, [encoding])` -- в строку (альтернатива `TextDecoder`),
- `readAsDataURL(blob)` -- в формат base64-кодированного URL.
```

```smart header="Для Web Worker также доступен `FileReaderSync`"
Для веб-воркеров доступен синхронный вариант `FileReader`, именуемый [FileReaderSync](https://www.w3.org/TR/FileAPI/#FileReaderSync).

Его методы считывания `read*` не генерируют события, а возвращают результат, как это делают обычные функции.

Но это только внутри веб-воркера, поскольку задержки в синхронных вызовах, которые возможны при чтении из файла, в веб-воркерах менее важны. Они не влияют на страницу.
```

## Итого

`File` объекты наследуют от `Blob`.

Помимо методов и свойств `Blob`, объекты `File` также имеют свойства `name` и `lastModified` плюс внутреннюю возможность чтения из файловой системы. Обычно мы получаем объекты `File` из пользовательского ввода, например, через `<input>` или перетаскиванием с помощью мыши, в событии `dragend`.

Объекты `FileReader` могут читать из файла или Blob в одном из трёх форматов:
- Строка (`readAsText`).
- `ArrayBuffer` (`readAsArrayBuffer`).
- URL в формате base64 (`readAsDataURL`).

Однако, во многих случаях нам не нужно читать содержимое файла. Как и в случае с Blob, мы можем создать короткий URL с помощью `URL.createObjectURL(file)` и использовать его в теге `<a>` или `<img>`. Таким образом, файл может быть загружен или показан в виде изображения, как часть canvas и т.д.

А если мы собираемся отправить `File` по сети, то это также легко, поскольку в сетевые методы, такие как `XMLHttpRequest` или `fetch`, встроена возможность отсылки `File`.
