<template>
    <div
        v-if="visible"
        ref="dropdownRef"
        class="ww-mention-dropdown"
        :style="dropdownStyles"
        @click.stop
    >
        <!-- Tabs -->
        <div class="ww-mention-dropdown__tabs" :style="tabsStyles">
            <button
                v-for="type in recordTypes"
                :key="type.id"
                class="ww-mention-dropdown__tab"
                :class="{ 'ww-mention-dropdown__tab--active': activeTab === type.id }"
                :style="getTabStyle(type.id)"
                @click="handleTabChange(type.id)"
            >
                {{ type.label }}
            </button>
        </div>

        <!-- Records list -->
        <div class="ww-mention-dropdown__list" :style="listStyles">
            <div
                v-if="filteredRecords.length === 0"
                class="ww-mention-dropdown__empty"
                :style="emptyStyles"
            >
                {{ filterText ? 'No results found' : 'No records available' }}
            </div>

            <div
                v-for="(record, index) in filteredRecords"
                :key="record.id"
                ref="recordRefs"
                class="ww-mention-dropdown__record"
                :class="{ 'ww-mention-dropdown__record--highlighted': index === highlightedIndex }"
                :style="getRecordStyle(index)"
                @click="handleRecordSelect(record)"
                @mouseenter="highlightedIndex = index"
            >
                <img
                    v-if="record.avatar"
                    :src="record.avatar"
                    class="ww-mention-dropdown__record-avatar"
                    alt=""
                />
                <span class="ww-mention-dropdown__record-label">{{ record.label }}</span>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, computed, watch, onMounted, onBeforeUnmount, nextTick } from 'vue';

export default {
    name: 'MentionDropdown',
    props: {
        visible: {
            type: Boolean,
            default: false,
        },
        recordTypes: {
            type: Array,
            default: () => [],
        },
        activeTab: {
            type: String,
            default: '',
        },
        filterText: {
            type: String,
            default: '',
        },
        highlightedIndex: {
            type: Number,
            default: 0,
        },
        position: {
            type: Object,
            default: () => ({ x: 0, y: 0 }),
        },
        // Styling props
        mentionDropdownBgColor: {
            type: String,
            default: '#ffffff',
        },
        mentionDropdownBorder: {
            type: String,
            default: '1px solid #e2e8f0',
        },
        mentionDropdownBorderRadius: {
            type: String,
            default: '12px',
        },
        mentionDropdownBoxShadow: {
            type: String,
            default: '0 4px 12px rgba(0, 0, 0, 0.15)',
        },
        mentionDropdownMaxHeight: {
            type: String,
            default: '300px',
        },
        mentionDropdownMaxWidth: {
            type: String,
            default: '400px',
        },
        mentionTabTextColor: {
            type: String,
            default: '#64748b',
        },
        mentionTabActiveColor: {
            type: String,
            default: '#3b82f6',
        },
        mentionTabHoverColor: {
            type: String,
            default: '#334155',
        },
        mentionRecordBgColor: {
            type: String,
            default: 'transparent',
        },
        mentionRecordHoverBgColor: {
            type: String,
            default: '#f1f5f9',
        },
        mentionRecordTextColor: {
            type: String,
            default: '#334155',
        },
        mentionRecordFontSize: {
            type: String,
            default: '0.875rem',
        },
    },
    emits: ['select-record', 'close', 'tab-change', 'update:highlighted-index'],
    setup(props, { emit }) {
        const dropdownRef = ref(null);
        const recordRefs = ref([]);

        const activeRecordType = computed(() => {
            return props.recordTypes.find(type => type.id === props.activeTab);
        });

        const filteredRecords = computed(() => {
            if (!activeRecordType.value || !activeRecordType.value.records) {
                return [];
            }

            const records = activeRecordType.value.records;
            if (!props.filterText) {
                return records;
            }

            const filter = props.filterText.toLowerCase();
            return records.filter(record => {
                return record.label && record.label.toLowerCase().includes(filter);
            });
        });

        const dropdownStyles = computed(() => ({
            backgroundColor: props.mentionDropdownBgColor,
            border: props.mentionDropdownBorder,
            borderRadius: props.mentionDropdownBorderRadius,
            boxShadow: props.mentionDropdownBoxShadow,
            maxWidth: props.mentionDropdownMaxWidth,
        }));

        const tabsStyles = computed(() => ({
            borderBottom: `1px solid ${props.mentionDropdownBorder.split(' ').pop()}`,
        }));

        const listStyles = computed(() => ({
            maxHeight: props.mentionDropdownMaxHeight,
        }));

        const emptyStyles = computed(() => ({
            color: props.mentionTabTextColor,
            fontSize: props.mentionRecordFontSize,
        }));

        const getTabStyle = (tabId) => {
            const isActive = tabId === props.activeTab;
            return {
                color: isActive ? props.mentionTabActiveColor : props.mentionTabTextColor,
                borderBottomColor: isActive ? props.mentionTabActiveColor : 'transparent',
            };
        };

        const getRecordStyle = (index) => {
            const isHighlighted = index === props.highlightedIndex;
            return {
                backgroundColor: isHighlighted ? props.mentionRecordHoverBgColor : props.mentionRecordBgColor,
                color: props.mentionRecordTextColor,
                fontSize: props.mentionRecordFontSize,
            };
        };

        const handleTabChange = (tabId) => {
            emit('tab-change', tabId);
            emit('update:highlighted-index', 0);
        };

        const handleRecordSelect = (record) => {
            emit('select-record', { recordTypeId: props.activeTab, record });
        };

        const handleClickOutside = (event) => {
            if (dropdownRef.value && !dropdownRef.value.contains(event.target)) {
                // Check if click is on mention button (it has its own handler)
                const isMentionButton = event.target.closest('.ww-chat-input-area__mention-btn');
                if (!isMentionButton) {
                    emit('close');
                }
            }
        };

        // Scroll highlighted item into view
        watch(
            () => props.highlightedIndex,
            async (newIndex) => {
                await nextTick();
                if (recordRefs.value && recordRefs.value[newIndex]) {
                    recordRefs.value[newIndex].scrollIntoView({
                        block: 'nearest',
                        behavior: 'smooth',
                    });
                }
            }
        );

        // Reset highlighted index when filter changes
        watch(
            () => props.filterText,
            () => {
                emit('update:highlighted-index', 0);
            }
        );

        // Reset highlighted index when filtered records change
        watch(
            filteredRecords,
            () => {
                if (props.highlightedIndex >= filteredRecords.value.length) {
                    emit('update:highlighted-index', Math.max(0, filteredRecords.value.length - 1));
                }
            }
        );

        onMounted(() => {
            // Use mousedown instead of click to catch the event before any blur handlers
            document.addEventListener('mousedown', handleClickOutside);
        });

        onBeforeUnmount(() => {
            document.removeEventListener('mousedown', handleClickOutside);
        });

        return {
            dropdownRef,
            recordRefs,
            filteredRecords,
            dropdownStyles,
            tabsStyles,
            listStyles,
            emptyStyles,
            getTabStyle,
            getRecordStyle,
            handleTabChange,
            handleRecordSelect,
        };
    },
};
</script>

<style lang="scss" scoped>
.ww-mention-dropdown {
    position: absolute;
    bottom: 100%;
    left: 0;
    margin-bottom: 8px;
    z-index: 1000;
    display: flex;
    flex-direction: column;
    animation: dropdown-fade-in 0.2s ease-out;

    &__tabs {
        display: flex;
        gap: 4px;
        padding: 8px 12px 0;
        overflow-x: auto;
        scrollbar-width: none;

        &::-webkit-scrollbar {
            display: none;
        }
    }

    &__tab {
        padding: 8px 16px;
        background: none;
        border: none;
        border-bottom: 2px solid transparent;
        cursor: pointer;
        font-size: 0.875rem;
        font-weight: 500;
        white-space: nowrap;
        transition: all 0.2s ease;

        &:hover {
            color: v-bind('mentionTabHoverColor');
        }

        &--active {
            font-weight: 600;
        }
    }

    &__list {
        overflow-y: auto;
        padding: 8px;
        scrollbar-width: thin;

        &::-webkit-scrollbar {
            width: 6px;
        }

        &::-webkit-scrollbar-track {
            background: transparent;
        }

        &::-webkit-scrollbar-thumb {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 3px;

            &:hover {
                background: rgba(0, 0, 0, 0.3);
            }
        }
    }

    &__empty {
        padding: 16px;
        text-align: center;
        font-style: italic;
        opacity: 0.7;
    }

    &__record {
        display: flex;
        align-items: center;
        gap: 12px;
        padding: 10px 12px;
        border-radius: 8px;
        cursor: pointer;
        transition: all 0.15s ease;

        &:hover {
            background-color: v-bind('mentionRecordHoverBgColor');
        }

        &--highlighted {
            background-color: v-bind('mentionRecordHoverBgColor');
        }
    }

    &__record-avatar {
        width: 32px;
        height: 32px;
        border-radius: 50%;
        object-fit: cover;
        flex-shrink: 0;
    }

    &__record-label {
        flex: 1;
        font-weight: 500;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
}

@keyframes dropdown-fade-in {
    from {
        opacity: 0;
        transform: translateY(4px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}
</style>
