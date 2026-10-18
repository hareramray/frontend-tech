# Internationalization and localization

[Curriculum](../README.md)

Prerequisites: [JavaScript](../13-javascript-fundamentals/tutorial.md), [semantic HTML](../03-semantic-html/tutorial.md), and [accessibility](../05-web-accessibility/tutorial.md).

## Learning outcomes

Separate translatable messages from program logic, format numbers and dates with explicit locale choices, handle plural categories and right-to-left presentation, and test beyond English. Internationalization creates the capability to support languages and regions; localization supplies the language, formatting, and cultural choices for a specific audience.

## Core concepts

A locale is not the same as a country, currency, time zone, or language preference. A Hindi-speaking user can live outside India and pay in another currency. Store these preferences separately when the product needs that distinction. Use a fallback chain for missing translations and let a user override inferred preferences.

The `Intl` family supports locale-sensitive formatting and comparisons. `NumberFormat`, `DateTimeFormat`, `PluralRules`, `RelativeTimeFormat`, `ListFormat`, `Collator`, and `Segmenter` solve different problems. Formatting a price does not convert its currency. Locale-sensitive number parsing is also a separate problem; `parseFloat` does not correctly parse every displayed localized number. [MDN Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)

## Worked tutorial: switch language and formatting

1. Save the following as `locale-lab/index.html` and open it in a modern browser. No installation is required.

```html
<!doctype html>
<html lang="en">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Locale lab</title>
<style>
  main { max-inline-size: 36rem; margin-inline: auto; padding: 1rem; }
  p { border-inline-start: .25rem solid teal; padding-inline-start: 1rem; }
</style>
<main>
  <h1 id="heading"></h1>
  <label for="locale">Language / اللغة</label>
  <select id="locale">
    <option value="en-US">English</option>
    <option value="ar-EG">العربية</option>
  </select>
  <label for="count" id="countLabel"></label>
  <input id="count" type="number" min="0" max="1000" step="1" value="3">
  <p id="summary" role="status"></p>
  <p id="price"></p>
  <p id="date"></p>
</main>
<script type="module">
  const messages = {
    en: {
      title: 'Study plan', count: 'Lessons',
      lessons: { one: '{n} lesson', other: '{n} lessons' },
    },
    ar: {
      title: 'خطة الدراسة', count: 'الدروس',
      lessons: {
        zero: 'لا دروس', one: 'درس واحد', two: 'درسان',
        few: '{n} دروس', many: '{n} درسًا', other: '{n} درس',
      },
    },
  };
  const localeSelect = document.querySelector('#locale');
  const countInput = document.querySelector('#count');
  function render() {
    const locale = localeSelect.value;
    const language = locale.startsWith('ar') ? 'ar' : 'en';
    const copy = messages[language];
    const count = Math.max(0, Math.min(1000, Math.floor(Number(countInput.value) || 0)));
    const category = new Intl.PluralRules(locale).select(count);
    const template = copy.lessons[category] ?? copy.lessons.other;
    document.documentElement.lang = language;
    document.documentElement.dir = language === 'ar' ? 'rtl' : 'ltr';
    document.title = copy.title;
    document.querySelector('#heading').textContent = copy.title;
    document.querySelector('#countLabel').textContent = copy.count;
    document.querySelector('#summary').textContent = template.replace(
      '{n}', new Intl.NumberFormat(locale).format(count)
    );
    document.querySelector('#price').textContent = new Intl.NumberFormat(locale, {
      style: 'currency', currency: 'INR',
    }).format(1250);
    document.querySelector('#date').textContent = new Intl.DateTimeFormat(locale, {
      dateStyle: 'full', timeZone: 'Asia/Kolkata',
    }).format(new Date('2026-01-15T12:00:00Z'));
  }
  localeSelect.addEventListener('change', render);
  countInput.addEventListener('input', render);
  render();
</script>
</html>
```

2. Switch between English and Arabic and try counts 0, 1, 2, 3, 11, and 100. Observe changes in plural wording, number presentation, and reading direction. The monetary amount remains INR 1250; only its display changes.
3. Resize the viewport and enlarge text. CSS logical properties move the border and padding with the writing direction. This is more robust than duplicating left/right styles throughout a codebase.

## Messages and data modeling

Keep complete messages together. Concatenating “You have” + number + “items” prevents translators from changing grammatical order. Production message systems often support ICU-style plural and select expressions, named placeholders, and translator context. Provide examples and explain what each variable means. Do not pass translated text to unsafe HTML insertion merely to support a link; use a message renderer that preserves safe component boundaries.

Store timestamps with an unambiguous representation and format them using the intended time zone. A birthday or calendar-only date has different semantics from an instant; interpreting it as midnight UTC can shift the displayed day for another zone. Sorting should use a suitable `Intl.Collator`, while searching may need explicit case, accent, and normalization policy.

## Advanced concerns and pitfalls

Use `dir="auto"` or `<bdi>` where user-supplied names may contain mixed-direction text. Mirror directional affordances deliberately, but do not blindly mirror logos, numbers, or every icon. Support line wrapping, longer translated labels, appropriate fonts, and input-method composition. Do not assume a JavaScript string length equals the number of user-perceived characters.

Server and client must agree on initial locale and time zone to avoid hydration differences. Load translations by locale without shipping all dictionaries to every visitor, but provide a useful fallback if a language chunk fails. Localized URLs need consistent navigation, canonical rules, and alternate-language metadata; see [SEO](../55-seo-and-structured-data/tutorial.md).

## Exercises and acceptance criteria

- Add a third locale with reviewed messages. Every plural category used by that locale must have an intentional fallback.
- Add a long pseudo-translation. No essential control may truncate into ambiguity or overlap adjacent content.
- Format the same instant in two explicit time zones. Explain any date change rather than treating it as a bug.
- Add a mixed-direction username using safe text insertion and directional isolation. Neighboring punctuation must remain understandable.

Primary references: [Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) and [W3C internationalization guidance](https://www.w3.org/International/). API guidance reviewed on 2026-10-18; production translations require language review.
