<template>
    <div class="wrapper" @keydown="tabHandler($event)" @keyup.ctrl="ctrlHandler">
        <div class="header">
            <span class="search-input">
                    <input  class="search" type="text" v-model="searchItem" v-bind:placeholder="i18n.searchPlaceholder"  v-autofocus="autofocus">
                    <i class="iconfont clear-icon" v-if="searchItem.length>0&&isSearch" @click.stop="cleanSearch">&#xe609;</i>
                     <i class="iconfont serach-icon" >&#xe60d;</i></span>
            <span @click="goHome" class="home"><i class="iconfont">&#xe601;</i></span>
        </div>
        <div class="main-content">
            <ul class="search-list" v-if="isSearch">
                <a v-for="(item,index) in searchList" class="list-item" v-if="searchList.length>0" @click.stop="openOnNewTab(item)" :class="{'selected':selectedId==item.id }">
                    <i class="order">{{index+1}}</i>
                    <i class="icon"><img :src="'chrome://favicon/' + item.url" ></i>
                    <span class="title" v-if="item.title">{{item.title}}</span>
                    <span class="title" v-if="!item.title">{{item.url}}</span>
                </a>
                <li class="no-reault" v-if="searchList.length==0">{{i18n.noMatchResult}}</li>
            </ul>
            <ul class="tree-list" v-if="!isSearch">
                <treeitem v-for="item in allNodes" class="folder-item" :folders="item" :key="item.id"></treeitem>
            </ul>
        </div>
    </div>
</template>
<script>
import treeitem from '../components/treeitem.vue'
export default {
    name: 'dashbord',
    data() {
        return {
            searchItem: '',
            allNodes: [],
            isSearch: false,
            searchList: [],
            selectedId: null,
            tabIndex: null,
            searchLen: 0,
            i18n: {
                searchPlaceholder: '',
                noMatchResult: ''
            }
        }
    },
    components: {
        treeitem: treeitem
    },
    created() {
        this.i18n.searchPlaceholder = chrome.i18n.getMessage("searchPlaceholder");
        this.i18n.noMatchResult = chrome.i18n.getMessage("noMatchResult");
        this.loadAllNodes();
    },
    directives: {
        autofocus: {
            inserted: function(el) {
                el.focus();
            },
            update: function(el) {
                el.focus();
            }
        }
    },
    methods: {
        tabHandler(e) {
            console.log(e)
            if (e.keyCode && (e.keyCode === 9 || e.keyCode === 13 || e.keyCode === 38 || e.keyCode === 40)) {
                e.preventDefault();
                e.stopPropagation();
                if (this.searchLen <= 0) {
                    return;
                }
                if (e.keyCode === 38 || e.keyCode === 40) {
                    if (e.keyCode === 38) {
                        this.tabIndex = (this.tabIndex - 1) >= 0 ? this.tabIndex - 1 : 0
                    } else if (e.keyCode === 40) {
                        this.tabIndex = (this.tabIndex + 1) <= this.searchLen - 1 ? this.tabIndex + 1 : this.searchLen - 1
                    }
                    let tabIndex = this.tabIndex;
                    this.selectedId = this.searchList[tabIndex].id;
                }
                // 判断enter事件
                if (e.keyCode === 13) {
                    this.openByTab(e)
                }
            }
        },
        ctrlHandler(e) {
            e.preventDefault();
            e.stopPropagation();
            if (this.searchLen <= 0) {
                return;
            }
            if (e.type === 'keyup' && e.key) {
                let tabIndex = parseInt(e.key);
                tabIndex = tabIndex >= 1 ? tabIndex : 1;
                tabIndex = tabIndex > 9 ? 9 : tabIndex;
                this.tabIndex = tabIndex - 1;
                this.selectedId = this.searchList[tabIndex - 1].id;
                this.openByTab(e)
                console.log(tabIndex)
                console.log(this.tabIndex)
                console.log(e)
            }
        },
        loadAllNodes() {
            this.allNodes = [];
            chrome.bookmarks.getTree((treeNodes) => {
                console.log(treeNodes);
                this.allNodes = treeNodes[0].children;
                console.log(this.allNodes);
            });
        },
        goHome() {
            chrome.tabs.create({
                'url': 'index.html',
                'selected': true
            });
        },
        searchBookmarks() {
            if (this.searchItem == '') {
                return;
            }
            this.isSearch = true;
            chrome.bookmarks.search(this.searchItem, (data) => {
                console.log(data)
                if (data && data.length > 0) {
                    this.searchList = data.filter(this.getTab);
                    this.selectedId = this.searchList[0].id;
                    this.tabIndex = 0;
                    this.searchLen = this.searchList.length;
                } else {
                    this.searchList = [];
                    this.selectedId = null;
                    this.searchLen = 0;
                }
            })
        },
        cleanSearch() {
            this.searchItem = '';
            this.isSearch = false;
        },
        openOnNewTab(item) {
            if (!item.url) {
                return;
            }
            let newTab = {
                url: item.url,
                active: true
            }
            console.log(newTab)
            chrome.tabs.create(newTab)
        },
        openByTab(e) {
            console.log(this.selectedId);
            if (this.searchLen == 0 || !this.selectedId) {
                return;
            }
            let tabIndex = this.tabIndex;
            let newTab = this.searchList[tabIndex];
            this.openOnNewTab(newTab);
        },
        getTab(item) {
            return item.url;
        }
    },
    watch: {
        searchItem: function(newVal) {
            if (newVal.length == 0) {
                console.log(newVal)
                this.selectedId = null;
                this.isSearch = false;
            } else {
                this.selectedId = null;
                this.searchBookmarks();
            }
        }
    }
}
</script>
<style lang="scss" scoped>
.wrapper {
    min-height: 100px;
    .header {
        display: flex;
        height: 30px;
        line-height: 30px;
        position: fixed;
        top: 0;
        padding: 5px;
        background: #f5f5f5;
        z-index: 500;
        .search-input {
            position: relative;
            .search {
                height: 24px;
                line-height: 24px;
                width: 340px;
                border: none;
                outline: none;
                padding: 3px 20px;
            }
            .serach-icon {
                position: absolute;
                left: 0;
                top: 0;
                color: #333;
                padding-left: 3px;
                cursor: pointer;
            }
            .clear-icon {
                position: absolute;
                right: 0;
                top: 0;
                color: #333;
                padding-right: 3px;
                cursor: pointer;
            }
        }
        .home {
            width: 30px;
            height: 30px;
            font-size: 20px;
            cursor: pointer;
        }
    }
    .main-content {
        width: 100%;
        height: auto;
        padding: 0 8px;
        margin-top: 40px;
        box-sizing: border-box;
        max-height: 560px;
        overflow-y: scroll;
        &::-webkit-scrollbar {
             width: 8px;
       }
        &::-webkit-scrollbar-track {
            background-color: #F6F6F6;
        }
        &::-webkit-scrollbar-button {
            display: none;
        }
        &::-webkit-scrollbar-thumb {
            border-radius: 5px;
            width: 6px;
            background-color: rgba(80,80,80,0.35);
        }
        .search-list {
            margin: 0;
            transition: all 0.5s;
            overflow-y:auto;
            .list-item {
                height: 28px;
                display: block;
                line-height: 28px;
                text-align: left;
                cursor: pointer;
                overflow: hidden;
                outline: none;
                padding-left: 15px;
                .order {
                    width: 8px;
                    text-align: center;
                    height: 28px;
                    line-height: 28px;
                    color: #42b983;
                }
                .icon {
                    padding-left: 5px;
                    width: 20px;
                    height: 28px;
                    line-height: 28px;
                    vertical-align: middle;
                    display: inline-block;
                    img {
                        padding-top: 5px;
                    }
                }
                .title {
                    padding-left: 5px;
                    height: 28px;
                    line-height: 28px;
                    color: #42b983;
                }
                &:hover {
                    background: #e4f1f9;
                }
                &.selected {
                    background: #e4f1f9;
                }
            }
            .no-reault {
                margin-top: 15px;
            }
        }
        .tree-list {
            margin: 0;
            transition: all 0.5s;
        }
    }
}
</style>