Why should we use react-table?

- Building your own table is hard

- Renders no UI elements, it's a table utility only, not a component

Columns are defined as an array of objects, with values 'header' and 'accessor'.

```js
export const COLUMNS = [
  {
    header: "id",
    accessor: "id",
  },
  {
    header: "First Name",
    accessor: "first_name",
  },
];
```

We use our columns like so:

```js
import React from "react";
import { useTable } from "react-table";
import MOCK_DATA from "./MOCK_DATA.json";
import { COLUMNS } from "./columns";

export const MyTable = () => {
  useTable({
    columns: COLUMNS,
    data: MOCK_DATA,
  });
  return <div></div>;
};
```

React-table recommends memoising the data for rows and columns, via the useMemo hook.

This ensures the data isn't recreated on every render. If we didn't do this, react-table would think it was receiving new data on every render and attempt to recalculate a lot of logic every time, which would negatively impact our performance.

```js
import React, { useMemo } from "react";
import { useTable } from "react-table";
import MOCK_DATA from "./MOCK_DATA.json";
import { COLUMNS } from "./columns";

export const MyTable = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  useTable({
    // can write it like this:
    columns: columns,
    data: data,
    // or like this, because of ES6 shorthand syntax:
    columns,
    data,
  });
  return <div></div>;
};
```

The call to useTable will return a table instance, which we will store in a constant.

```js
const tableInstance = useTable({
  columns,
  data,
});
```

We then define a basic table structure using HTML.

```js
export const MyTable = () => {
  const columns = useMemo(() => COLUMNS, []);
  const data = useMemo(() => MOCK_DATA, []);

  useTable({
    columns,
    data,
  });
  return (
    <table>
      <thead>
        <tr>
          <th></th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td></td>
        </tr>
      </tbody>
    </table>
  );
};
```

We then destructure some properties from our table instance.

```js
const { getTableProps, getTableBodyProps, headerGroups, rows, prepareRow } =
  tableInstance;
```

These are functions and arrays that the useTable() hook has given to us to allow for easy table creation. We use all of these with our HTML for react-table to work as intended.

getTableProps is a function which needs to be destructured on the <table> tag.

```js
<table {...getTableProps()}>
```

Similarly, we destructure getTableBodyProps on the <tbody> tag.

```js
<tbody {...getTableBodyProps()}>
```

headerGroups is an array, which contains the column heading information for inside the <thead> tag.

```js
<thead>
  {
    headerGroups.map((headerGroup) => (
      <tr {...headerGroup.getHeaderGroupProps()}>
        <th>
          {
            headerGroup.headers.map((column) => (
              <th {...column.getHeaderProps()}>{column.render('Header')}</th>
            ))
          }
        </h>
      </tr>
    ))
  }
</thead>
```

Rows go inside the <tbody> tag.

```js
<tbody>
  {rows.map((row) => (
    <tr {...row.getRowProps()}>
      {row.cells.map((cell) => (
        <td {...cell.getCellProps()}>{cell.render("Cell")}</td>
      ))}
    </tr>
  ))}
</tbody>
```

## Adding a footer to a table:

Similar to how we defined a Header property for each column, we also define a Footer.

```js
export const COLUMNS = [
  {
    Header: "First Name",
    Footer: "First Name",
    accessor: "first_name",
  },
];
```

We have to add footerGroups to our destructured properties we're getting from react-table.

```js
const {
  getTableProps,
  getTableBodyProps,
  headerGroups,
  footerGroups,
  rows,
  prepareRow,
} = useTable({
  columns,
  data,
});
```

Then we add a <tfoot> tag after the body.

```js
<tfoot>
  {footerGroups.map((footerGroup) => (
    <tr {...footerGroup.getFooterGroupProps()}>
      {footerGroup.headers.map((column) => (
        <td {...column.getFooterProps}>{column.render("Footer")}</td>
      ))}
    </tr>
  ))}
</tfoot>
```

This will render a footer at the bottom of the table with the same titles as our header.

## Header Groups

By default, we will have separate individual columns. Sometimes, we might need to group a few columns into a common heading.

```js
export const COLUMNS = [];

export const GROUPED_COLUMNS = [
  {
    Header: "Id",
    Footer: "Id",
    accessor: "id",
  },
  {
    Header: "Name",
    Footer: "Name",
    columns: [
      {
        Header: "First Name",
        Footer: "First Name",
        accessor: "first_name",
      },
      {
        Header: "Last Name",
        Footer: "Last Name",
        accessor: "last_name",
      },
    ],
  },
];

const columns = useMemo(() => GROUPED_COLUMNS, []);
```

Changing to the above would result in a single Id column, and both first and last name grouped within a parent 'Name' column.

## Sorting

```js
const {
  getTableProps,
  getTableBodyProps,
  headerGroups,
  footerGroups,
  rows,
  prepareRow,
} = useTable(
  {
    columns,
    data,
  },
  useSortBy
);

<thead>
  {headerGroups.map((headerGroup) => (
    <tr {...headerGroup.getHeaderGroupProps()}>
      {headerGroup.headers.map((column) => (
        // this adds properties related to the sorting feature to the individual column
        <th {...column.getHeaderProps(column.getSortByToggleProps())}>
          {column.render("Header")}
          // we can now add icons representing the sorted state of the column
          <span>{column.isSorted ? (column.isSortedDesc ? '🔽' : '🔼') : ''}
        </th>
      ))}
    </tr>
  ))}
</thead>;
```

## Formatting

Note react-table can format dates, but it needs them provided in the proper ISO format. What if we want to format our dates to be displayed in a user-friendly way?

We'd use the date-fns package.

The Cell property controls what is rendered in the UI. It receives a function.

```js
import {format} from 'date-fns'

{
  Header: 'Date of birrth',
  Footer: 'Date of birth',
  accessor: 'date_of_birth',
  // first arg = value,
  // second arg = format in which you want the date displayed

  // we destructure 'value' from the data provided to our table
  Cell: ({value}) => {return format{new Date(value), 'dd/MM/yyyy'}}
}

```

## Global Filtering

We can have filters which apply to all columns at once, e.g. searching.

```js
// export const GlobalFilter = ({filter, setFilter}) => {
//   return (
//     <span>
//       Search: {' '}
//       <input value={filter || ''}
//       onChange=((e) => ))/>
//     </span>
//   )
// }
```
