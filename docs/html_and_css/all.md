## 1. HTML: структура и теги

HTML-документ состоит из дерева элементов, каждый элемент задаётся тегами `<имя>…</имя>` или одиночными `<имя />`.

### 1.1. Основная структура

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Заголовок страницы</title>
</head>
<body>
<!-- Контент -->
</body>
</html>
```

### 1.2. Списки тегов

1. **Метаданные (в `<head>`)**

    * `<meta>` — описание документа, charset, viewport.
    * `<title>` — заголовок вкладки.
    * `<link>` — подключение CSS, фавикон.
    * `<script>` — подключение JS.
    * `<style>` — встроенные стили.

2. **Структурные**

    * `<header>`, `<nav>`, `<main>`, `<section>`, `<article>`, `<aside>`, `<footer>` — семантика разделов.
    * `<div>` — контейнер без семантики.
    * `<span>` — строчный контейнер.

3. **Текстовые**

    * Заголовки: `<h1>`…`<h6>`.
    * Параграфы: `<p>`.
    * Ссылки: `<a href="…">`.
    * Выделение: `<strong>`, `<em>`, `<b>`, `<i>`, `<mark>`, `<small>`, `<del>`, `<ins>`.
    * Цитаты: `<blockquote>`, `<q>`; `cite`-атрибут.
    * Предформатированный: `<pre>`, `<code>`, `<kbd>`, `<samp>`.

4. **Медиа**

    * Изображения: `<img src="…" alt="…" />`.
    * Видео/аудио: `<video controls>`, `<audio controls>`.
    * `<picture>`, `<source>` для адаптивных изображений.

5. **Таблицы**

    * `<table>`, `<thead>`, `<tbody>`, `<tfoot>`, `<tr>`, `<th>`, `<td>`, `<caption>`.

6. **Формы**

    * `<form action="…" method="…">`
    * Поля: `<input>` (тип `text`, `password`, `checkbox`, `radio`, `file`, `email`, `number`, `date`, `submit` и др.),
    * `<textarea>`, `<select>`, `<option>`, `<label>`, `<fieldset>`, `<legend>`.

7. **Интерактивные**

    * `<button>`, `<details>`, `<summary>`, `<dialog>`, `<meter>`, `<progress>`.

---

## 2. CSS: синтаксис и способы подключения

CSS управляет стилями элементов. Синтаксис:

```css
/* Селектор { свойство: значение; … } */
h1 {
    font-size: 2rem;
    color: navy;
}
```

### 2.1. Способы подключения

1. **Внешний файл**

   ```html
   <link rel="stylesheet" href="styles.css">
   ```
2. **Встроенный**

   ```html
   <style>
     /* стили */
   </style>
   ```
3. **Inline-стили**

   ```html
   <div style="color: red; margin: 10px;">…</div>
   ```

---

## 3. Селекторы CSS

1. **По тегу**: `div { … }`
2. **По классу**: `.className { … }`
3. **По идентификатору**: `#idName { … }`
4. **Комбинации**:

    * Подчинённость: `ul li { … }`
    * Дочерний: `ul > li { … }`
    * Соседний: `h1 + p { … }`
    * Общий сосед: `h1 ~ p { … }`
5. **Атрибутные**: `a[target="_blank"] { … }`
6. **Псевдоклассы**:

    * Состояния: `:hover`, `:active`, `:focus`
    * Позиции в списке: `:first-child`, `:last-child`, `:nth-child(2n)`
    * Формы: `:checked`, `:disabled`
7. **Псевдоэлементы**:

    * `::before`, `::after`, `::first-letter`, `::first-line`, `::placeholder`.

---

## 4. Блочная и строчная модель

* **`display`**:

    * `block` ( `<div>`, `<p>`, `<h1>` )
    * `inline` ( `<span>`, `<a>`, `<strong>` )
    * `inline-block`
    * `flex`, `grid`
    * `none`

* **Box Model**:

    * `content` (ширина/высота)
    * `padding`
    * `border`
    * `margin`

---

## 5. Флексбокс (Flexbox)

```css
.container {
    display: flex;
    flex-direction: row | column;
    justify-content: flex-start | center | space-between | space-around;
    align-items: flex-start | center | stretch;
    flex-wrap: nowrap | wrap;
}

.item {
    flex: 0 1 auto; /* grow, shrink, basis */
}
```

---

## 6. CSS Grid

```css
.grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: auto 200px;
    gap: 10px 20px; /* row-gap column-gap */
}

.grid-item {
    grid-column: 1 / 3; /* start / end */
    grid-row: 2 / 3;
}
```

---

## 7. Адаптивность и медиа-запросы

```css
@media (max-width: 768px) {
    .container {
        flex-direction: column;
    }
}
```

---

## 8. Позиционирование

* `position: static | relative | absolute | fixed | sticky;`
* `top`, `right`, `bottom`, `left`, `z-index`

---

## 9. Цвета и шрифты

* Цвета: `#RRGGBB`, `rgb()`, `rgba()`, `hsl()`, `transparent`, названия.
* Шрифты:

  ```css
  font-family: "Arial", sans-serif;
  font-size: 16px | 1rem;
  font-weight: normal | bold | 400 | 700;
  line-height: 1.5;
  ```

---

## 10. Анимации и переходы

* **Transitions**:

  ```css
  .box {
    transition: all 0.3s ease-in-out;
  }
  .box:hover { transform: scale(1.1); }
  ```
* **Keyframes**:

  ```css
  @keyframes slide {
    from { transform: translateX(0); }
    to   { transform: translateX(100px); }
  }
  .animate { animation: slide 2s infinite alternate; }
  ```
