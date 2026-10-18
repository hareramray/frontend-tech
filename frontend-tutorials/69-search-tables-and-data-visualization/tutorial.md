# 69. Search, tables, and data visualization

[Curriculum](../README.md)

**Prerequisites:** [collections](../16-arrays-collections-and-immutability/tutorial.md), [accessibility](../05-web-accessibility/tutorial.md), and [server state](../35-server-state-and-caching/tutorial.md).

**You will learn to:** separate source data from view state, implement accessible sorting, and choose between local and server-side data operations.

## Define the data contract

A table combines records with view state: query, filters, sort order, pagination, selection, and visible columns. Keep these concepts separate. Sorting should not mutate your canonical data accidentally; filtering should not permanently delete records. Preserve stable row IDs so selection survives reordering.

Client-side processing works well when the complete dataset is small enough to download, store, and process. Server-side search is required when data is large, access-controlled, or frequently changing. Filtering one downloaded page does not search the entire dataset. Make that distinction explicit in the interface.

## Tutorial: searchable, sortable table

Place this body fragment in the [HTML shell](../SETUP.md) and run it locally:

```html
<h1>Course catalog</h1>
<label for="query">Filter course names</label>
<input id="query" type="search">
<p id="count" role="status"></p>
<table>
  <caption>Available courses</caption>
  <thead><tr>
    <th id="name-heading" scope="col" aria-sort="ascending">
      <button id="sort">Course: toggle name sort</button>
    </th>
    <th scope="col">Lessons</th>
  </tr></thead>
  <tbody id="rows"></tbody>
</table>
<script type="module">
const courses = [
  { id: 'a', name: 'Accessibility', lessons: 8 },
  { id: 'b', name: 'Browser APIs', lessons: 12 },
  { id: 'c', name: 'CSS layout', lessons: 10 }
];
const query = document.querySelector('#query');
const rows = document.querySelector('#rows');
const heading = document.querySelector('#name-heading');
const count = document.querySelector('#count');
const collator = new Intl.Collator('en', { sensitivity: 'base', numeric: true });
let direction = 1;
function render() {
  const term = query.value.trim().toLocaleLowerCase('en');
  const visible = courses
    .filter(course => course.name.toLocaleLowerCase('en').includes(term))
    .sort((a, b) => direction * collator.compare(a.name, b.name));
  const fragment = document.createDocumentFragment();
  for (const course of visible) {
    const row = document.createElement('tr');
    const name = document.createElement('th');
    name.scope = 'row';
    name.textContent = course.name;
    const lessons = document.createElement('td');
    lessons.textContent = String(course.lessons);
    row.append(name, lessons);
    fragment.append(row);
  }
  rows.replaceChildren(fragment);
  count.textContent = `${visible.length} courses found.`;
  heading.setAttribute('aria-sort', direction === 1 ? 'ascending' : 'descending');
}
query.addEventListener('input', render);
document.querySelector('#sort').addEventListener('click', () => {
  direction *= -1;
  render();
});
render();
</script>
```

Expected result: typing `css` leaves one row; activating the sort button reverses alphabetical order. Focus remains on the persistent control because only the table body is replaced. All source records remain unchanged. The comparator uses locale-aware comparison, although this simple substring search does not implement accent folding, stemming, typo tolerance, or full-text ranking.

## Scale search and selection

For remote search, cancel obsolete requests or ignore their responses, debounce where helpful, and expose loading, empty, and error states separately. Include query, filters, sort, and page identity in cache keys. Reset pagination when a filter changes. Cursor pagination handles some changing-data scenarios better than numeric offsets, but requires stable server ordering and an explicit cursor contract.

Decide whether “select all” means the visible page or all matching results. For server-wide selection, represent the query plus exclusions rather than downloading every ID. Authorization must be checked again when a bulk action executes. Shareable filters belong in URL parameters when they are non-sensitive.

Virtualization renders only a visible window to reduce DOM work. It complicates focus, screen-reader navigation, variable row heights, printing, and browser find. Prefer ordinary pagination when it meets the need. A semantic table is not an ARIA grid; a grid carries additional keyboard interaction obligations.

## Visualize without distorting

Choose a chart based on the question: lines for ordered trends, bars for category comparison, scatterplots for relationships. Label units, time zones, missing values, and aggregation windows. Avoid implying causation from correlation. A truncated bar axis can exaggerate differences, while a log scale needs clear labeling.

Provide text summaries and accessible data alternatives. Color should not be the only encoding. Dense interactive charts need keyboard navigation, meaningful focus, and discoverable tooltips. Exported CSV should preserve data types where possible and consider spreadsheet formula injection when user-controlled cells begin with formula characters.

**Exercise:** add lesson-count sorting and URL-backed filters. Acceptance criteria: reloading preserves the filter, sorting is numeric for lesson counts, an empty result explains what happened, back navigation restores state, and keyboard users can identify the active sort order.

**References:** [WAI sortable table example](https://www.w3.org/WAI/ARIA/apg/patterns/table/examples/sortable-table/), [Intl.Collator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Collator), [D3 documentation](https://d3js.org/getting-started).
