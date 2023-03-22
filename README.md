# Создание форм с помощью Formik в React

## Cодержание
- Что такое Formik
- Почему стоит использовать Formik
- Управление состоянием формы
- Валидация и сообщения об ошибках в Formik
  - Как показать сообщения об ошибках в Formik
  - Валидация с помощью Yup
- Отправка формы
- Заключение

## Что такое Formik

`Formik` — это легкая, бесплатная библиотека с открытым исходным кодом для `React` или `React Native`, которая решает три основные задачи при создании форм:

 - Управление состоянием формы
 - Валидация формы
 - Отправка формы

[Джаред Палмер](https://twitter.com/jaredpalmer) написал `Formik` при создании большой внутренней административной панели. Цель состояла в том, чтобы стандартизировать не только все формы, но и то, как через них проходят данные, что значительно упростило бы тестирование, рефакторинг и анализ этих форм. 

## Почему стоит использовать Formik

Создание формы только на `React` потребует написать каждую часть процесса от настройки состояния до отправки формы самостоятельно:
- Нужно будет хранить и отслеживания значения полей формы, ошибки и статус валидации каждого поля
- потребуется также обрабатывать ввод пользователя и менять состояние формы соответственно
- необходимо будет продумать и самостоятельно описать валидации 
- описать процесс отправки формы

`Formik` берет на себя все выше перечисленные заботы и предоставляет ряд компонентов и хуков для создания форм в `React` и `React Native`. Далее в статье мы познакомимся с основными компоненты этой библиотеки: `<Formik />`, `<Form />`, `<Field />`, и `<ErrorMessage />`.

## Управление состоянием формы

Для того, чтобы начать работать с `Formik` достаточно обернуть форму в компонент `<Formik />:`

```jsx
<Formik>
  <Form>
    {/* здесь остальной код */}
  </Form>
</Formik>
```

С помощью пропса **initialValues** инициализируется начальное состояние формы внутри `Formik`, а чтобы получать значения и обновлять состояние полей формы, вместо обычного HTML-элемента `<input />` можно воспользоваться компонентом `<Field />`. Этот компонент будет самостоятельно синхронизировать состояние, и нам не потребуется передавать через пропсы `value` и `onChange`:

```js
<Formik
  initialValues={{ email: "", password: "" }}
  onSubmit={({ setSubmitting }) => {
    console.log("Form is validated! Submitting the form...");
    setSubmitting(false);
  }}
>
  {() => (
    <Form>
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <Field
          type="email"
          name="email"
          className="form-control"
        />
      </div>
      <div className="form-group">
        <label htmlFor="password">Password</label>
        <Field
          type="password"
          name="password"
          className="form-control"
        />
      </div>
    </Form>
  )}
</Formik>

```

У `Formik` есть собственный метод **handleChange**, который можно использовать для обновления состояния при пользовательском вводе данных в поля формы, таким образом отпадает необходимость реализовывать собственный метод `handleChange`. В примере ниже метод `handleChange` обновит значения поля `email` в состоянии формы на основании атрибута `name`, если в этом поле будут изменения:


```js
const SignupForm = () => {
  const formik = useFormik({
    initialValues: {
      email: "",
    },
    onSubmit: (values) => {
      console.log(JSON.stringify(values, null, 2));
    },
  });
  return (
    <form onSubmit={formik.handleSubmit}>
      <label htmlFor="email">Email Address</label>
      <input
        id="email"
        name="email"
        type="email"
        onChange={formik.handleChange}
        value={formik.values.email}
      />

      <button type="submit">Submit</button>
    </form>
  );
};
```

Таким образом, метод `handleChange` используется с `input` элементами, а компонент `<Field>` самостоятельно обновляет значения без необходимости реализации метода `handleChange`. 

## Валидация и сообщения об ошибках в Formik

Валидация очень важна при создании форм. Если формы не обрабатываются должным образом, это может привести к большому количеству ошибок.

Формы должны сообщать пользователям, какие поля являются обязательными, какие типы значений разрешены в определенных полях. Это также помогает дать пользователям четкое представление о том, что не так с их вводом.

Валидация в `Formik` выполняется автоматически во время определенных событий. Все распространенные события, такие как ввод данных пользователем, изменение фокуса и отправка, охватываются, и нам не нужно о них беспокоиться. 
`Formik` поддерживает валидацию как на уровне формы, так и на уровне поля. Все, что нужно сделать, это передать функцию проверки в компонент `<Formik />` или `<Field />` через проп `validate`:

```jsx
// Synchronous validation
 const validate = (values, props) => {
   const errors = {};
 
   if (!values.email) {
     errors.email = 'Required';
   } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}$/i.test(values.email)) {
     errors.email = 'Invalid email address';
   }
 
   //...
 
   return errors;
 };
```

### Как показать сообщения об ошибках в Formik

`Formik` предоставляет компонент `<ErrorMessage />` для автоматического отображения сообщений об ошибках  для компонента `<Field />` с соответствующим именем (`name`). 

`<ErrorMessage />` через свойства позволяет указать какой HTML-тег будет отображаться в итоге: 

```js
<Field
  type="email"
  name="email"
  className={`form-control ${
    touched.email && errors.email ? "is-invalid" : ""
  }`}
/>
<ErrorMessage
  component="div"
  name="email"
  className="invalid-feedback"
/>
```

### Валидация с помощью Yup

Как мы говорили выше, `Formik` поддерживает валидацию как на уровне формы, так и на уровне поля, причем за нами остается право написать свою собственную реализацию валидации. Однако можно пойти дальше и воспользоваться возможностью связать `Formik` c `Yup`.

[Yup](https://github.com/jquense/yup) - это библиотека для валидации объектов `js`. Она позволяет описать схему валидации объекта, где для каждого свойства объекта устанавливается ряд ограничений, а далее на этом объекте вызывается метод валидации `validate`.

`Formik` предоставляет поддержку схем валидации `Yup`, которая называется `validationSchema`, она автоматически преобразует  ошибки валидации из `Yup` в объект, ключи которого соответствуют значениям полей формы.

Для того чтобы добавить `Yup` в проект, нужно установить его из npm:

```bash
npm install yup --save
```

```js
import React from 'react';
import { Formik, Form, Field } from 'formik';
import * as Yup from 'yup';

const SignupSchema = Yup.object().shape({
  firstName: Yup.string()
    .min(2, 'Минимум 2 буквы')
    .max(50, 'Максимум 50 букв')
    .required('Обязательное поле'),
  lastName: Yup.string()
    .min(2, 'Минимум 2 буквы')
    .max(50, 'Максимум 50 букв')
    .required('Обязательное поле'),
  email: Yup.string().email('Неверный email').required('Обязательное поле'),
});

export const ValidationSchemaExample = () => (
  <div>
    <h1>Signup</h1>
    <Formik
      initialValues={{
        firstName: '',
        lastName: '',
        email: '',
      }}
      validationSchema={SignupSchema}
      onSubmit={ (values) => {
        console.log(values);
      }}
    >
      {({ errors, touched }) => (
        <Form>
          <Field name="firstName" />
          {errors.firstName && touched.firstName ? (
            <div>{errors.firstName}</div>
          ) : null}
          <Field name="lastName" />
          {errors.lastName && touched.lastName ? (
            <div>{errors.lastName}</div>
          ) : null}
          <Field name="email" type="email" />
          {errors.email && touched.email ? <div>{errors.email}</div> : null}
          <button type="submit">Submit</button>
        </Form>
      )}
    </Formik>
  </div>
);
```

## Отправка формы

`Formik` использует функцию `onSubmit` для обработки данных формы всякий раз, когда нажимается кнопка отправки. Валидация запускается автоматически, и отправка формы отменится, если есть какие-либо ошибки.

В примере ниже функция `onSubmit` вызывается при отправке формы и использует функцию `setSubmitting` для обновления состояния компонента `Formik` в процессе отправки.

```js
 <Formik
   initialValues={{ username: "", email: "", password: "" }}
   // . . .
   onSubmit={(values, { setSubmitting }) => {
     setTimeout(() => {
       console.log(JSON.stringify(values, null, 2));
       setSubmitting(false);
     }, 400);
   }}
>
```

## Заключение

`Formik` является одной из тех библиотек с открытым исходным кодом, которая необходима, если вы пишете много форм в своем приложении `React`. Она делает процесс создания формы относительно быстрым и интуитивно понятным, особенно при создании сложных форм. 

В этой статье мы рассмотрели основные возможности библиотеки. Если вы хотите узнать больше о `Formik`, то вы можете найти дополнительную информацию в [официальной документации](https://formik.org/).
