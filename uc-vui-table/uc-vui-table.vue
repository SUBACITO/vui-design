<template>
	<div class="overflow-x-auto border rounded-lg">
		<div style="min-width: 1200px;">
			<!-- HEADER -->
			<div class="flex bg-blue-darken-1 text-white px-3 py-2 font-medium items-center">
				<!-- Checkbox col -->
				<div class="w-10 flex justify-center flex-shrink-0" v-if="itemKey">
					<v-checkbox-btn :model-value="isAllSelected" :indeterminate="isIndeterminate" color="white"
						density="compact" hide-details @update:model-value="toggleAll" />
				</div>
				<!-- STT col -->
				<div class="flex-shrink-0" style="width: 48px;">STT</div>
				<!-- Data cols -->
				<div v-for="col in headers" :key="col.key" :style="getColStyle(col)">
					{{ col.title }}
				</div>
			</div>

			<v-alert density="compact" variant="tonal" color="info">
				<template v-slot:prepend>
					<v-icon size="small">mdi-file-document-multiple-outline</v-icon>
				</template>
				<div class="text-caption text-center">
					Hệ thống tìm thấy <strong>{{ pagination.TotalRows || items.length }}</strong> hồ sơ phù hợp
					<span v-if="selectedIds.size > 0 && itemKey" class="ms-2 font-weight-bold text-primary">
						— Đã chọn {{ selectedIds.size }}
					</span>
				</div>
			</v-alert>

			<v-virtual-scroll ref="virtualRef" :items="items" :item-height="48" class="overflow-auto"
				:style="customHeight">
				<template #default="{ item, index }">
					<div class="flex px-3 py-2 border-b items-center">
						<!-- Checkbox -->
						<div class="w-10 flex justify-center flex-shrink-0" v-if="itemKey">
							<v-checkbox-btn :model-value="selectedIds.has(item[itemKey])" color="primary"
								density="compact" hide-details @update:model-value="toggleItem(item)" />
						</div>
						<!-- STT -->
						<div class="flex-shrink-0" style="width: 48px;">
							{{ getIndex(index) }}
						</div>
						<!-- DATA -->
						<div v-for="col in headers" :key="col.key" :style="getColStyle(col)">
							<slot :name="col.key" :item="item">
								{{ item[col.key] }}
							</slot>
						</div>
					</div>
				</template>
			</v-virtual-scroll>

			<!-- FOOTER HEADER -->
			<div class="flex bg-blue-darken-1 text-white px-3 py-2 font-medium">
				<!-- Checkbox placeholder -->
				<div class="w-10 flex-shrink-0" v-if="itemKey"></div>
				<!-- STT placeholder -->
				<div class="flex-shrink-0" style="width: 48px;">STT</div>
				<!-- Data cols -->
				<div v-for="col in headers" :key="col.key" :style="getColStyle(col)">
					{{ col.title }}
				</div>
			</div>

			<div class="d-flex justify-end align-center ga-3 pa-4">
				<div class="text-subtitle-2 text-grey">
					<v-icon size="16" class="me-1">mdi-book-open-page-variant-outline</v-icon>
					Trang: {{ pagination.PageNumber }} <span class="mx-1 text-grey-lighten-1">|</span> {{
					pagination.TotalPages }}
				</div>

				<v-btn variant="outlined" color="primary" size="small" class="text-none" append-icon="mdi-arrow-right"
					:disabled="pagination.PageNumber >= pagination.TotalPages"
					@click="changePage(pagination.PageNumber + 1)">
					Tiếp tục
				</v-btn>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		props: {
			items: Array,
			loading: Boolean,
			total: Number,
			pagination: {
				type: Object,
				default: () => ({ "TotalRows": 0, "TotalPages": 0, "PageNumber": 0, "PageSize": 0 })
			},
			customHeight: {
				type: String,
				default: "height: calc(100vh - 260px)"
			},
			listSelected: Array,
			itemKey: "",
			headers: {
				type: Array,
				default: () => ([])
			}
		},
	
		data() {
			return {
				selectedIds: new Set(),
			}
		},
		computed: {
			isAllSelected() {
				return this.items?.length > 0 &&
					this.items.every(i => this.selectedIds.has(i[this.itemKey]))
			},
			isIndeterminate() {
				const count = this.items.filter(i =>
					this.selectedIds.has(i[this.itemKey])
				).length
	
				return count > 0 && count < this.items.length
			}
		},
		watch: {
			listSelected(newValue) {
				// Check close dialog => clear selected items
				if (newValue?.length == 0) {
					this.selectedIds.clear()
				}
			}
		},
		methods: {
			getColStyle(col) {
				// Nếu col có width cố định → dùng width + flex-shrink: 0 (giống v-data-table)
				if (col.width) {
					return {
						width: typeof col.width === 'number' ? `${col.width}px` : col.width,
						minWidth: typeof col.width === 'number' ? `${col.width}px` : col.width,
						flexShrink: 0,
					}
				}
				// Không có width → flex-1 với minWidth (cột co giãn)
				return {
					flex: 1,
					minWidth: col.minWidth || '150px',
					overflow: 'hidden',
				}
			},
			toggleItem(item) {
				const key = item[this.itemKey]
	
				const updated = new Set(this.selectedIds)
	
				if (updated.has(key)) {
					updated.delete(key)
				} else {
					updated.add(key)
				}
	
				this.selectedIds = updated
				this.emitSelected()
			},
			toggleAll(val) {
				if (val) {
					this.selectedIds = new Set(
						this.items.map(i => i[this.itemKey])
					)
				} else {
					this.selectedIds = new Set()
				}
	
				this.emitSelected()
			},
			emitSelected() {
				const selectedRows = this.items.filter(i =>
					this.selectedIds.has(i[this.itemKey])
				)
	
				this.$emit('update:listSelected', selectedRows)
			},
			scrollToTop() {
				this.$nextTick(() => {
					const el = this.$refs.virtualRef?.$el
					if (!el) return
	
					// Vuetify 3: scroll nằm ở root
					el.scrollTop = 0
				})
			},
			scrollToBottom() {
				this.$nextTick(() => {
					requestAnimationFrame(() => {
						const el = this.$refs.virtualRef?.$el
						if (!el) return
	
						el.scrollTop = el.scrollHeight
					})
				})
			},
			getIndex(index) {
				return index + 1
			},
			formatDate(date) {
				return moment(date).format('DD/MM/YYYY')
			},
			changePage(page) {
				this.$emit('update:pagination', {
					...this.pagination,
					page
				})
			}
		}
	}
</script>