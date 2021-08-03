
# Тип даних Символ

За специфікацією, ключами об’єкта можуть бути рядок, або символ. Не цифри, не логічні значення, лише рядки або символи, тільки ці два типи.

До цього часу ми використовували лише рядки. А тепер давайте подивимося, які переваги можуть дати нам символи.

## Символи

"Символ" являє собою унікальний ідентифікатор.

Створити символ можна за допомогою `Symbol()`:

```js
// id - це новостворенний символ
let id = Symbol();
```

Після створення символу ми можемо надати йому опис (так званe ім’я символу), в основному це корисно для процесу налагодження:

```js
// Створюємо символ id з описом (іменем) "id"
let id = Symbol("id");
```

Символи гарантовано будуть унікальними. Навіть якщо ми створюємо багато символів з однаковим описом, вони мають різні значення. Опис - це просто мітка, яка ні на що не впливає.

Наприклад, ось два символи з однаковим описом -- вони не рівні:

```js run
let id1 = Symbol("id");
let id2 = Symbol("id");

*!*
alert(id1 == id2); // false
*/!*
```

Якщо ви знайомі з Ruby чи іншою мовою програмування, яка також має «символи» - будь ласка, не думайте що це те саме. Символи в JavaScript мають свої особливості.

````warn header="Символи не перетворюються автоматично в рядок"
Більшість значень у JavaScript підтримують неявне перетворення в рядок. Наприклад, ми можемо помістити майже будь-яке значення в `alert`, і воно автоматично перетворить його в рядок. Символи - вони особливі. Вони не перетворюються автоматично.

Наприклад, цей `alert` видасть помилку:

```js run
let id = Symbol("id");
*!*
alert(id); // TypeError: Cannot convert a Symbol value to a string
*/!*
```

Це "захист" мови від плутанини, оскільки рядки та символи принципово відрізняються і не повинні неконтрольовано перетворюватись з одного в інше.

Якщо ми дійсно хочемо відобразити символ, нам потрібно явно перетворити його за допомогою `.toString()`, ось так:
```js run
let id = Symbol("id");
*!*
alert(id.toString()); // Symbol(id), тепер все працює
*/!*
```

Або викликати властивість `symbol.description` щоб відобразити тільки опис:
```js run
let id = Symbol("id");
*!*
alert(id.description); // id
*/!*
```

````

## "Приховані" властивості

Символи дозволяють нам створювати "приховані" властивості об’єкта, до яких жодна інша частина програми не може випадково отримати доступ або перезаписати їх.

Наприклад, якщо ми працюємо з `user` об’єктами, які належать сторонньому коду. Ми хотіли б додати до них ідентифікатори.

Використаємо для цього ключ символа:

```js run
let user = { // належить сторонньому коду
  name: "John"
};

let id = Symbol("id");

user[id] = 1;

alert( user[id] ); // ми можемо отримати доступ до даних, використовуючи символ як ключ
```

Яка користь від використання `Symbol("id")` замість рядка `"id"`?

Оскільки об’єкт `user` належить сторонньому коду, і цей код також працює з ними, нам не варто просто додавати до нього будь-які поля. Це небезпечно. Але до символу неможливо отримати доступ випадково, сторонній код, ймовірно, навіть не побачить його, тому, додавання поля до об’єкта не викличе ніяких проблем.

Крім того, уявіть, що інший скрипт хоче мати власний ідентифікатор всередині `user` для своїх цілей. Це може бути ще одна бібліотека JavaScript, так що сценарії абсолютно не знають один про одного.

Тоді цей скрипт може створити свій власний `Symbol("id")`, як ось цей:

```js
// ...
let id = Symbol("id");

user[id] = "Їхній ідентифікатор";
```

Конфлікту не буде між нашим та їх ідентифікаторами, оскільки символи завжди різні, навіть якщо вони мають однакову назву.

...Але якби ми використовували рядок `"id"` замість символу з тією ж метою, тоді *виникне* конфлікт

```js
let user = { name: "Тарас" };

// Наш скрипт використовує ключ "id"
user.id = "Наш ідентифікатор";

// ...Інший скрипт теж хоче використовувати ключ "id" для своїх цілей...

user.id = "Їхній ідентифікатор"
// Ой! Властивість перезаписана стороннім скриптом!
```

### Символи в літералі об’єкта

Якщо ми хочемо використовувати символ у літералі об’єкта `{...}`, нам потрібні обгорнути його в квадратні дужки.

Ось так:

```js
let id = Symbol("id");

let user = {
  name: "Тарас",
*!*
  [id]: 123 // просто "id": 123 не спрацює
*/!*
};
```
Це тому, що нам потрібно використовувати значення змінної `id` як ключ, а не рядок "id".

### Символи ігноруються циклом for..in

Властивості в якості символів не перебираються `for..in` циклом.

Наприклад:

```js run
let id = Symbol("id");
let user = {
  name: "Тарас",
  age: 30,
  [id]: 123
};

*!*
for (let key in user) alert(key); // name, age (дані ключі не є символами)
*/!*

// але працює прямий доступ за допомогою символу
alert( "Прямий доступ: " + user[id] );
```

`Object.keys(user)` також ігнорує їх. Це частина загального принципу "приховування символьних властивостей". Якщо інший скрипт або бібліотека спробує перебрати наш об’єкт за допомогою цикла, він несподівано не отримає доступ до символьних властивостей.

А ось, [Object.assign](mdn:js/Object/assign) копіює властивості рядка та символу:

```js run
let id = Symbol("id");
let user = {
  [id]: 123
};

let clone = Object.assign({}, user);

alert( clone[id] ); // 123
```

Тут немає парадокса. Саме так задумано. Ідея полягає в тому, що коли ми клонуємо об’єкт або об’єднуємо об’єкти, ми зазвичай хочемо скопіювати *всі* властивості (включаючи і властивості з ключами в якості символу, як наприклад `id` з прикладу вище).

## Глобальні символи

Як ми вже бачили, зазвичай усі символи унікальні, навіть якщо вони мають однакову назву. Але іноді ми хочемо, щоб однойменні символи були однаковими сутностями. Наприклад, різні частини нашого додатку хочуть отримати доступ до символу `"id"`, що означає абсолютно однакову властивість.

Для цього існує *глобальний реєстр символів*. Ми можемо створити в ньому символи і отримати до них доступ пізніше, і це гарантує, що повторні звернення з тим самим іменем нам повернуть абсолютно однаковий символ.

Для того, щоб знайти (створити, якщо його немає) символ у реєстрі, використовуйте `Symbol.for(key)`.

Цей виклик перевіряє глобальний реєстр, і якщо є символ з іменем `key`, тоді повертає його, інакше створює новий символ `Symbol(key)` і зберігає його в реєстрі за вказаним `key`.

Наприклад:

```js run
// шукаємо в глобального реєстрі
let id = Symbol.for("id"); // якщо такого символу немає, він буде створений

// шукаємо знову, але присвоюємо в іншу змінну (можливо в іншій частині коду)
let idAgain = Symbol.for("id");

// як бачимо, це один і той самий символ
alert( id === idAgain ); // true
```

Символи всередині реєстру називаються *глобальні символи*. Якщо вам потрібні символи, доступні скрізь у коді - використовуйте глобальні символи.

```smart header="Схоже як в Ruby"
У деяких мовах програмування, таких як Ruby, для одного імені використовується один символ. Не можуть існувати різні символи з однаковими іменами.

Як бачимо, в JavaScript, це правило теж працює, але тільки для глобальних символів.
```

### Symbol.keyFor

Для глобальних символів, не тільки `Symbol.for(key)` повертає символ за іменем, також існує протилежний метод: `Symbol.keyFor(sym)`, який працює наоборот: приймає глобальний символ і повертає його ім’я.

Наприклад:

```js run
// отримуємо символ за іменем
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// отримуємо ім’я за символом
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

`Symbol.keyFor` внутрішньо використовує глобальний реєстр символів для пошуку ключа символу. Тож це не працює для неглобальних символів. Якщо символ не є глобальним, він не зможе його знайти і повернем нам `undefined`.

Тим не менш, будь-які символи мають властивість `description`.

Наприклад:

```js run
let globalSymbol = Symbol.for("name");
let localSymbol = Symbol("name");

alert( Symbol.keyFor(globalSymbol) ); // name, глобальний символ
alert( Symbol.keyFor(localSymbol) ); // undefined, не глобальний сивол

alert( localSymbol.description ); // name
```

## Системні символи

Існує багато “системних” символів, які JavaScript використовує внутрішньо, і ми можемо використовувати їх для налаштування різних аспектів наших об’єктів.

Вони вказані в таблиці специфікації [Well-known symbols](https://tc39.github.io/ecma262/#sec-well-known-symbols):

- `Symbol.hasInstance`
- `Symbol.isConcatSpreadable`
- `Symbol.iterator`
- `Symbol.toPrimitive`
- ...та інші.

До прикладу, `Symbol.toPrimitive` дозволяє описати  правила для об’єкта до примітивного перетворення. Ми побачимо його використання дуже скоро.

З іншими системними символами познайомимось ближче, коли ми будемо вивчати відповідні мовні особливості.

## Підсумки

`Symbol`-- це примітивний тип даних який використовується для унікальних ідентифікаторів.

Символ створюється за допомогою виклику `Symbol()` з необов’язковим описом (ім’я).

Символи - завжди унікальні, навіть якщо вони мають однакову назву. Якщо ми хочемо, щоб однойменні символи були рівними за значенням, тоді слід використовувати глобальний реєстр: `Symbol.for(key)` повертає (створює за потреби) глобальний символ з `key` в якості імені. 
Декілька викликів `Symbol.for` з тією ж `key` повертає той самий символ.

Символи мають два основних варіанти використання:

1. "Приховані" властивості об’єкта.
    Якщо ми хочемо додати властивість до об’єкта, який «належить» іншому скрипту або бібліотеці, ми можемо створити символ і використовувати його як ключ. Властивість у вигляді символу не відображається в `for..in`, тому вона не буде випадково оброблена разом з іншими властивостями. Крім того, він не буде безпосередньо доступний, оскільки інший скрипт не має нашого символу. Таким чином власність буде захищена від випадкового використання або перезапису.

    Тож ми можемо ховати щось потрібне для нас в об’єкти, що сторонні не повинні бачити, використовуючи символічні властивості.

2. JavaScript використовує багато системних символів, доступних як `Symbol.*`. Ми можемо використовувати їх для зміни деяких вбудованих форм поведінки. До прикладу, пізніше в підручнику ми будемо використовувати `Symbol.iterator` для [ітераторів](info:iterable), `Symbol.toPrimitive` для налаштування [перетворення об’єкта в примітив](info:object-toprimitive) та інше.

Технічні символи не приховані на 100%. Існує вбудований метод [Object.getOwnPropertySymbols(obj)](mdn:js/Object/getOwnPropertySymbols) що дозволяє отримати всі символи. Також існує метод з іменем [Reflect.ownKeys(obj)](mdn:js/Reflect/ownKeys) що повертає *всі* ключі об’єкта, включаючи символьні. Тож вони насправді не зовсім приховані. Але більшість бібліотек, вбудованих функцій та конструкцій синтаксису не використовують ці методи.