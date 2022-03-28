# Useful Resources
- PlantUML used in ZORA README
# Typescript

# Interfaces 

```typescript
export interface Props {
	title: string
	color?: string
}```

# Types
Difference is that it works with unions

```typescript
type Props = {
	title: string
	color?: string
}

type Age = string | number
```

# Generics
Think of generics as a placeholder for type

```typescript
function getArray<T>(items: T[]): T[] {
	return new Array().concat(items);
}

let numArray = getArray<number>([1,2,3,4])
let strArray = getArray<string>(['brad','john']

numArray.push('hello') // returns error with generics, valid if using any
```

