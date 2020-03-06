<template>
<ul class="c-list">
    <li v-for="page in pages"
        :key="page.id"
        class="c-list__item-h"
    >
        <Page ref="pageComponent"
              :default-page-id="defaultPageId"
              :page="page"
              :page-title="pageTitle"
        />
    </li>
</ul>
</template>

<script>
import { EVENT_DELETE_PAGE, EVENT_RENAME_PAGE, EVENT_UPDATE_PAGE, EVENT_SELECT_PAGE, TOGGLE_NAV } from '../notebook-constants';
import { deleteNotebookEntries } from '../utils/notebook-entries';
import { getDefaultNotebook, clearDefaultNotebook } from '../utils/notebook-storage';
import Page from './page-component.vue';

export default {
    inject: ['openmct'],
    components: {
        Page
    },
    props: {
        defaultPageId: {
            type: String,
            default() {
                return '';
            }
        },
        domainObject: {
            type: Object,
            default() {
                return {};
            }
        },
        pages: {
            type: Array,
            required: true,
            default() {
                return [];
            }
        },
        sections: {
            type: Array,
            required: true,
            default() {
                return [];
            }
        },
        pageTitle: {
            type: String,
            default() {
                return '';
            }
        },
        sidebarCoversEntries: {
            type: Boolean,
            default() {
                return false;
            }
        }
    },
    data() {
        return {
        }
    },
    watch: {
        pages(newpages) {
            this.removeAllListeners();
            this.addListeners();
        }
    },
    mounted() {
        this.addListeners();
    },
    destroyed() {
        this.removeAllListeners();
    },
    methods: {
        addListeners() {
            setTimeout(() => {
                if (this.$refs.pageComponent) {
                    this.$refs.pageComponent.forEach(element => {
                        element.$on(EVENT_DELETE_PAGE, this.deletePage);
                        element.$on(EVENT_RENAME_PAGE, this.updatePage);
                        element.$on(EVENT_SELECT_PAGE, this.selectPage);
                    });
                }
            },0);
        },
        deletePage(id) {
            const selectedSection = this.sections.find(s => s.isSelected);
            const page = this.pages.filter(p => p.id !== id);
            deleteNotebookEntries(this.openmct, this.domainObject, selectedSection, page);

            const selectedPage = this.pages.find(p => p.isSelected);
            const defaultNotebook = getDefaultNotebook();
            const defaultpage = defaultNotebook && defaultNotebook.page;
            const isPageSelected = selectedPage && selectedPage.id === id;
            const isPageDefault = defaultpage && defaultpage.id === id;
            const pages = this.pages.filter(s => s.id !== id);

            if (isPageSelected && defaultpage) {
                pages.forEach(s => {
                    s.isSelected = false;
                    if (defaultpage && defaultpage.id === s.id) {
                        s.isSelected = true;
                    }
                });
            }

            if (isPageDefault) {
                clearDefaultNotebook();
            }

            if (pages.length && isPageSelected && (!defaultpage || isPageDefault)) {
                pages[0].isSelected = true;
            }

            this.$parent.$emit(EVENT_UPDATE_PAGE, { pages });
        },
        removeAllListeners() {
            if (this.$refs.pageComponent) {
                this.$refs.pageComponent.forEach(element => {
                    element.$off();
                });
            }
        },
        selectPage(id) {
            const pages = this.pages.map(page => {
                const isSelected = page.id === id;
                page.isSelected = isSelected;

                return page;
            });

            this.$parent.$emit(EVENT_UPDATE_PAGE, { pages, id });

            // Add test here for whether or not to toggle the nav
            if (this.sidebarCoversEntries) {
                this.$emit(TOGGLE_NAV);
            }
        },
        updatePage(newPage) {
            const pages = this.pages.map(page =>
                page.id === newPage.id
                    ? newPage
                    : page);
            this.$parent.$emit(EVENT_UPDATE_PAGE, { pages });
        }
    }
}
</script>