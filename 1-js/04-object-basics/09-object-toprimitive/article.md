
# Перетворення об’єктів в примітиви

Що відбувається, коли об’єкти додаються `obj1 + obj2`, віднімаються `obj1 - obj2` або друкуються за допомогою `alert(obj)`?

JavaScript не дозволяє налаштувати, як працюють оператори з об’єктами. На відміну від деяких інших мов програмування, таких як Ruby або C++, ми не можемо реалізувати спеціальний об’єктний метод для обробки додавання (або інших операторів).

У разі таких операцій об’єкти автоматично перетворюються на примітиви, а потім операція здійснюється над цими примітивами та повертає результат у вигляді примітивного значення.

Це важливе обмеження, оскільки результат `obj1 + obj2` не може бути іншим об’єктом!

Наприклад, ми не можемо зробити об’єкти, що представляють вектори або матриці (або досягнення або що завгодно) та додати їх і очікувати, що "підсумковий" результат буде об’єктом. Автоматично такі архітектурні особливості недоступні.

Отже, оскільки ми не можемо автоматично за допомогою мови програмування виконувати подібні операції над об’єктами, то в реальних проєктах немає "математики з об’єктами". Коли стаються подібні операції (наприклад, `obj1 + obj2`), причиною цьому зазвичай є помилка програмування.

У цьому розділі ми розглянемо те, як об’єкти перетворюється на примітиви і як налаштувати це.

У нас є два цілі:

1. Це дозволить нам зрозуміти, що відбувається у випадку помилок коду, коли така операція відбулася випадково.
2. Є винятки, де такі операції можливі і доцільні. Наприклад, віднімання або порівняння дати (`Date` об’єкти). Ми будемо зустрічатися з ними пізніше.

## Правила перетворення

У розділі <info:type-conversions> ми бачили правила для перетворення чисел, рядків та булевих примітивів. Але ми залишили пробіл для об’єктів. Тепер, оскільки ми знаємо про методи та символи, можемо заповнити його.

1. Всі об’єкти -- це `true` в булевому контексті. Є лише числові та рядкові конверсії.
2. Числове перетворення відбувається, коли ми віднімаємо об’єкти або застосовуємо математичні функції. Наприклад, `Date` об’єкти (розглянуті в розділі <info:date>) можуть відніматися, і результат `date1 - date2` -- це різниця у часі між двома датами.
3. Що стосується перетворення рядків -- це зазвичай відбувається, коли ми виводимо об’єкт, наприклад `alert(obj)` і в подібних контекстах.

Ми можемо точно налагоджувати перетворення рядків та чисел, використовуючи спеціальні методи об’єкта.

Є три варіанти перетворення типів, що відбуваються в різних ситуаціях.

Вони називаються "підказками" ("hints"), як описано в [специфікації](https://tc39.github.io/ecma262/#sec-toprimitive):

`"string"`
: Перетворення об’єкта в рядок відбувається коли ми виконуємо операцію, яка очікує рядок, над об’єктом. Наприклад, `alert`:

    ```js
    // вивід
    alert(obj);

    // використання об’єкта як ключа властивості об’єкта
    anotherObj[obj] = 123;
    ```

`"number"`
: Перетворення об’єкта в число, коли ми робимо математичні операції:

    ```js
    // явне перетворення
    let num = Number(obj);

    // математичні операції (крім бінарного додавання)
    let n = +obj; // унарне додавання
    let delta = date1 - date2;

    // порівняння менше/більше
    let greater = user1 > user2;
    ```

`"default"`
: Виникає в рідкісних випадках, коли оператор "не впевнений", який тип очікується.

    Наприклад, бінарний плюс `+` може працювати як з рядками (об’єднувати їх), так і з цифрами (додавати їх), тому обидва випадки -- рядки та цифри -- будуть працювати. Отже, якщо бінарний плюс отримує об’єкт як аргумент, він використовує підказку `"default"`, щоб перетворити його.

    Також, якщо об’єкт порівнюється (`==`) з рядком, числом чи символом, тоді незрозуміло, яке порівняння використати, тому використовується підказка `"default"`.

    ```js
    // бінарний плюс використовує підказку "default"
    let total = obj1 + obj2;

    // obj == цифра використовує підказку "default"
    if (user == 1) { ... };
    ```

    Оператори порівняння більше та менше, такі як `<` `>`, також можуть працювати як з рядками, так і з цифрами. Проте, вони з історичних причин використовують `"number"` підказку, а не `"default"`.

    На практиці, хоча нам не потрібно пам’ятати ці особливі деталі, тому що всі вбудовані об’єкти, крім одного випадку (об’єкт `Date`, ми дізнаємося пізніше) реалізовують `"default"` перетворення так само, як `"number"`. І ми можемо зробити те ж саме.

```smart header="Відсутність підказки `\"boolean\"`"
Будь ласка, зверніть увагу -- є лише три підказки. Це просто.

Немає "boolean" підказки (всі об’єкти `true` у булевому контексті) або будь-яких ще. І якщо ми однаково ставимося до `default` і `number`, як і більшість вбудованих об’єктів, то є лише два перетворення.
```

** Щоб зробити перетворення, JavaScript намагається знайти та викликати три методи об’єкта: **

1. Викликати `obj[Symbol.toPrimitive](hint)` -- метод з символьним ключем `Symbol.toPrimitive` (системний символ), якщо такий метод існує,
2. Інакше, якщо підказка - це `"string"`
    - спробує `obj.toString()` та `obj.valueOf()` -- будь-що, що існує.
3. Інакше, якщо підказка - `"номер"` або `"default"`
    - спробує `obj.valueOf()` та `obj.toString()` -- будь-що, що існує.

## Symbol.toPrimitive

Почнемо з першого методу. Є вбудований символ під назвою `Symbol.toPrimitive`, який слід використовувати для назви методу перетворення, як наприклад:

```js
obj[Symbol.toPrimitive] = function(hint) {
  // тут йде код, щоб перетворити цей об’єкт в примітив
  // він повинен повернути примітивне значення
  // hint = один з "string", "number", "default"
};
```

Якщо метод `symbol.toprimitive` існує, він використовується для всіх підказок, і не потрібно більше методів.

Наприклад, тут об’єкт `user` реалізує його:

```js run
let user = {
  name: "Іван",
  money: 1000,

  [Symbol.toPrimitive](hint) {
    alert(`hint: ${hint}`);
    return hint == "string" ? `{name: "${this.name}"}` : this.money;
  }
};

// демонстрація перетворення:
alert(user); // hint: string -> {name: "Іван"}
alert(+user); // hint: number -> 1000
alert(user + 500); // hint: default -> 1500
```

Як ми бачимо з коду, `user` стає самоописаним рядком або грошовою сумою залежно від перетворення. Єдиний метод `[Symbol.toPrimitive]` об’єкту `user` обробляє всі випадки перетворення.


## toString/valueOf

Якщо немає `Symbol.toPrimitive` тоді JavaScript намагається знайти методи `toString` і `valueOf`:

- Для "string" підказки: `tostring`, і якщо цей метод не існує, то `valueOf` (таким чином `toString` має пріоритет при перетворенні в рядок).
- Для інших підказок: `valueOf`, і якщо це не існує, то `toString` (таким чином `valueOf` має пріоритет для математики).

Методи `toString` і `valueOf` походять з давніх часів. Вони не є символами (багато часу назад символи не існували), а скоріше є "звичними" методами, що названі за допомогою рядків. Вони забезпечують альтернативний шлях "старого стилю" для реалізації перетворення.

Ці методи повинні повертати примітивне значення. Якщо `toString` чи `valueOf` повертає об’єкт, то він ігнорується (так само, якби цього методу не існувало).

За замовчуванням, простий об’єкт має слідувати методами `toString` та `valueOf`:

- Метод `toString` повертає рядок `"[object Object]"`.
- Метод `valueOf` повертає сам об’єкт.

Ось демо:

```js run
let user = {name: "Іван"};

alert(user); // [object Object]
alert(user.valueOf() === user); // true
```

Отже, якщо ми спробуємо використовувати об’єкт як рядок, наприклад в `alert` та ін., то за замовчуванням ми бачимо `[object Object]`.

За замовчуванням метод `valueOf` згадується тут лише заради повноти, щоб уникнути будь-якої плутанини. Як ви бачите, він повертає сам об’єкт, і тому ігнорується. Не питайте мене, чому це для історичних причин. Тому ми можемо припустити, що цього не існує.

Реалізуємо ці методи для налаштування перетворення.

Наприклад, тут `user` робить те ж саме, що й вище, використовуючи комбінацію `toString` і `valueOf` замість `Symbol.toPrimitive`:

```js run
let user = {
  name: "Іван",
  money: 1000,

  // для hint="string"
  toString() {
    return `{name: "${this.name}"}`;
  },

  // для hint="number" чи "default"
  valueOf() {
    return this.money;
  }

};

alert(user); // toString -> {name: "Іван"}
alert(+user); // valueOf -> 1000
alert(user + 500); // valueOf -> 1500
```

Як ми бачимо, поведінка така ж, як і в попередньому прикладі з `Symbol.toPrimitive`.

Часто ми хочемо, щоб в одному місці перехоплювалися та оброблялися всі перетворення в примітиви. У цьому випадку ми можемо реалізувати `toString`, як наприклад:

```js run
let user = {
  name: "Іван",

  toString() {
    return this.name;
  }
};

alert(user); // toString -> Іван
alert(user + 500); // toString -> Іван500
```

За відсутності `Symbol.toPrimitive` і `valueOf`, `toString` буде обробляти всі примітивні перетворення.

### Перетворення може повернути будь-який примітивний тип

Важливо знати про всі методи примітивні перетворення те, що вони не обов’язково повертають "підказаний" примітив.

Немає контролю, чи повертає `toString` саме рядок, або чи `symbol.toprimitive` метод повертає номер для підказки `"number"`.

Єдина обов’язкова річ: ці методи повинні повертати примітивний тип, а не об’єкт.

```smart header="Історичні нотатки"
З історичних причин, якщо `toString` чи `valueOf` повертає об’єкт. В цьому немає помилки, але таке значення ігнорується (так само, якби цей метод не існував). Це тому, що в давнину в JavaScript не було хорошого концепту "помилка".

Навпаки, `Symbol.toPrimitive` *повинен* повернути примітив, інакше буде помилка.
```

## Подальші перетворення

Як ми вже знаємо, багато операторів та функцій виконують перетворення типу, наприклад, множення `*` перетворює операнди в цифри.

Якщо ми передамо об’єкт як аргумент, то є два етапи:
1. об’єкт перетворюється на примітив (використовуючи правила, описані вище).
2. Якщо отриманий примітив не є правильним типом, він перетворюється.

Наприклад:

```js run
let obj = {
  // toString обробляє всі перетворення за відсутності інших методів
  toString() {
    return "2";
  }
};

alert(obj * 2); // 4, об’єкт перетворився на примітив "2", потім множення зробило це числом
```

1. Множення `obj * 2` спочатку перетворює об’єкт в примітив (це рядок `"2"`).
2. Тоді `"2" * 2` стає `2 * 2` (рядок перетворюється на номер).

Бінарний плюс буде об’єднувати рядки в такій же ситуації, оскільки він приймає рядки:

```js run
let obj = {
  toString() {
    return "2";
  }
};

alert(obj + 2); // 22 ("2" + 2), перетворення до примітиву повернуло рядок => Конкатенація
```

## Підсумки

Об’єктно-примітивне перетворення викликається автоматично багатьма вбудованими функціями та операторами, які очікують примітиву як значення.

Є 3 типи (підказки) цього:
- `"string"` (для `alert` та інших операцій, які потребують рядка)
- `"number"` (для математичних операцій)
- `"default"` (кілька операторів)

Специфікація явно описує, який оператор використовує яку підказку. Є дуже мало операторів, які "не знаю, що очікувати" і використовують підказку `"default"`. Зазвичай для вбудованих об’єктів `"default"` підказка обробляється так само, як `"number"`, тому на практиці останні дві часто об’єднуються разом.

Алгоритм перетворення це:

1. Викликати `obj[Symbol.toPrimitive](hint)`, якщо метод існує,
2. Інакше, якщо підказка -- це `"string"`
    - спробувати `obj.toString()` та `obj.valueOf()`, залежно від того, що існує.
3. Інакше, якщо підказка -- це `"number"` чи `"default"`
    - спробувати `obj.valueOf()` та `obj.toString()`, залежно від того, що існує.

На практиці досить часто для реалізації достатньо лише `obj.toString()` як методу для перетворення рядків, який повинен повернути "читабельне для людини" представлення об’єкта, для цілей логування або пошуку помилок.

Що стосується математичних операцій, JavaScript не забезпечує способу "перевизначити" їх використання, тому реальні проєкти рідко використовують математичні операції на об’єктах.