```
<template>
	<div>
		<handson-table :hotSettings="hotSettings" />
	</div>
</template>

<script>
	export default {
		data() {
			return {
				hotSettings: {
					data: [
						{ company: 'Tagcat', country: 'United Kingdom', rating: 4.4 },
						{ company: 'Zoomzone', country: 'Japan', rating: 4.5 },
						{ company: 'Meeveo', country: 'United States', rating: 4.6 },
					],
					columns: [
						{ data: 'company', title: 'Company', width: 100 },
						{
							data: 'country', title: 'Country', width: 170, type: 'dropdown', source: ['United Kingdom', 'Japan', 'United States']
						},
						{ data: 'rating', title: 'Rating', width: 100, type: 'numeric' },
					]
				}
			}
		}
	}
</script>
```
