<template>
    <div class="kiwi-container" v-bind:class="{
            /* 'kiwi-container-' + bufferType: true, */
            'kiwi-container--sidebar-open': sidebarOpen,
            'kiwi-container--no-sidebar': buffer && !buffer.isChannel(),
            'kiwi-container--mini': isHalfSize,
    }">
        <template v-if="buffer">
            <div @click.stop="toggleStateBrowser" class="kiwi-container-toggledraw-statebrowser">
                <i class="fa fa-bars" aria-hidden="true"></i>
            </div>
            <container-header :buffer="buffer"></container-header>
            <div @click.stop="toggleSidebar" v-bind:class="{
                'kiwi-container-toggledraw-sidebar': true,
                'kiwi-container-toggledraw-sidebar--disabled': !buffer.isChannel()
            }">
                <i class="fa fa-ellipsis-v" aria-hidden="true"></i>
            </div>

            <template v-if="buffer.isServer()">
                <server-view :network="network" :buffer="buffer"></server-view>
            </template>
            <template v-else>
                <sidebar
                    v-if="buffer.isChannel()"
                    :network="network"
                    :buffer="buffer"
                    :users="users"
                ></sidebar>
                <message-list :buffer="buffer" :messages="messages" :users="users"></message-list>
            </template>
        </template>
        <template v-else>
            {{$t('container_welcome')}}
            <a @click.stop="toggleStateBrowser">{{$t('container_statebrowser')}}</a>
        </template>
    </div>
</template>

<script>

import state from 'src/libs/state';
import ContainerHeader from './ContainerHeader';
import Sidebar from './Sidebar';
import MessageList from './MessageList';
import ServerView from './ServerView';

export default {
    components: {
        ContainerHeader,
        Sidebar,
        MessageList,
        ServerView,
    },
    data: function data() {
        return {
            sidebarOpen: false,
        };
    },
    props: ['network', 'buffer', 'users', 'messages', 'isHalfSize'],
    computed: {
        bufferType: function bufferType() {
            let type = '';

            if (!this.buffer) {
                type = 'none';
            } else if (this.buffer.isServer()) {
                type = 'server';
            } else if (this.buffer.isChannel()) {
                type = 'channel';
            } else if (this.buffer.isQuery()) {
                type = 'query';
            }

            return type;
        },
    },
    methods: {
        toggleStateBrowser: function toggleStateBrowser() {
            state.$emit('statebrowser.toggle');
        },
        toggleSidebar: function toggleSidebar() {
            if (this.buffer.isChannel()) {
                state.$emit('sidebar.toggle');
            }
        },
    },
    created: function created() {
        this.listen(state, 'sidebar.toggle', () => {
            state.$emit('sidebar.' + (this.sidebarOpen ? 'hide' : 'show'));
        });
        this.listen(state, 'sidebar.show', () => {
            this.sidebarOpen = true;
        });
        this.listen(state, 'sidebar.hide', () => {
            this.sidebarOpen = false;
        });
    },
};
</script>

<style>

.kiwi-container {
    box-sizing: border-box;
    display: flex;
    flex-direction: column;
}
.kiwi-header {
    margin-right: 200px;
    z-index: 1;
}

.kiwi-sidebar {
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    width: 200px;
    z-index: 2;
}
.kiwi-messagelist {
    margin-right: 200px;
    flex: 1;
}
.kiwi-serverview {
    flex: 1;
}

.kiwi-container--no-sidebar .kiwi-header,
.kiwi-container--no-sidebar .kiwi-messagelist {
    margin-right: 0;
}

.kiwi-container-toggledraw-statebrowser,
.kiwi-container-toggledraw-sidebar {
    display: none;
    width: 50px;
    position: absolute;
    top: 0;
    height: 50px;
    box-sizing: border-box;
}

.kiwi-container-toggledraw-statebrowser {
    left: 0;
}
.kiwi-container-toggledraw-sidebar {
    right: 0;
}


@media screen and (max-width: 850px) {
    .kiwi-header {
        margin-left: 50px;
        margin-right: 50px;
    }
    .kiwi-sidebar {
        right: -200px;
        top: 50px;
    }
    .kiwi-messagelist {
        margin-right: 0;
    }
    .kiwi-container-toggledraw-statebrowser,
    .kiwi-container-toggledraw-sidebar {
        display: block;
    }

    .kiwi-container--sidebar-open .kiwi-sidebar {
        right: 0;
    }

}

</style>
