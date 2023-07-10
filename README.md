# Epic Stack Time Zone Client Hint Example

This example adds a new
[client hint](https://github.com/epicweb-dev/epic-stack/blob/main/docs/client-hints.md)
to get the user's time zone. This is helpful when rendering dates and times on
the server. By returning the correct local time, we eliminate the "flash of
incorrect content" when the server and local times are not the same.

## Changes

_app/utils/client-hints.tsx_

```ts
export const clientHints = {
	// ...
	timeZone: {
		cookieName: 'CH-time-zone',
		getValueCode: `Intl.DateTimeFormat().resolvedOptions().timeZone`,
		fallback: 'UTC',
		transform(value: string | null) {
			return value ?? 'UTC'
		},
	},
	// add other hints here
}
```

_app/utils/misc.tsx_

```ts
export function getDateTimeFormat(
	request: Request,
	options?: Intl.DateTimeFormatOptions,
) {
	const locales = parseAcceptLanguage(request.headers.get('accept-language'), {
		validate: Intl.DateTimeFormat.supportedLocalesOf,
	})
	const locale = locales[0] ?? 'en-US'

	// change your default options here
	const defaultOptions: Intl.DateTimeFormatOptions = {
		year: 'numeric',
		month: 'numeric',
		day: 'numeric',
		hour: 'numeric',
		minute: 'numeric',
	}
	options = {
		...defaultOptions,
		...options,
		timeZone: options?.timeZone ?? getHints(request).timeZone ?? 'UTC',
	}
	return new Intl.DateTimeFormat(locale, options)
}
```

_root.tsx_

```ts
// default options to use timeZone client hint
const localDateTimeFormat = getDateTimeFormat(request)
const serverDateTimeFormat = getDateTimeFormat(request, {
	timeZone: 'UTC', // override timezone here
	month: 'short', // override month style, etc.
})
const now = new Date()

return json({
	serverTime: serverDateTimeFormat.format(now),
	localTime: localDateTimeFormat.format(now),
})
```
