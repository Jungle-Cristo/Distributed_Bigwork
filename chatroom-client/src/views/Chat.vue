<template>
  <div class="chat-layout">
    <!-- Left Sidebar -->
    <div class="sidebar">
      <div class="sidebar-header">
        <div class="user-info">
          <div class="user-avatar" @click="showProfile = true" title="个人资料">
            <img v-if="userStore.user?.avatar && !userStore.user.avatar.includes('default')" :src="userStore.user.avatar" class="avatar-img" />
            <span v-else>{{ (userStore.nickname || userStore.username || '?')[0] }}</span>
          </div>
          <div class="user-details">
            <div class="nickname">{{ userStore.nickname || userStore.username }}</div>
            <div class="status-indicator">
              <span class="status-dot"></span>
              <span>在线</span>
            </div>
          </div>
          <el-dropdown trigger="click" @command="handleUserCommand">
            <span class="dropdown-trigger">
              <el-icon><ArrowDown /></el-icon>
            </span>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item command="profile">个人设置</el-dropdown-item>
                <el-dropdown-item command="background">聊天背景</el-dropdown-item>
                <el-dropdown-item command="deleteAccount" divided>注销账户</el-dropdown-item>
                <el-dropdown-item command="logout" divided>退出登录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
        </div>
        <div class="header-actions">
          <el-badge :value="contactStore.pendingRequests.length" :hidden="!contactStore.hasPendingRequests">
            <button class="action-btn" @click="openFriendRequests">
              <el-icon><UserFilled /></el-icon>
            </button>
          </el-badge>
          <button class="action-btn" @click="showAddFriend = true">
            <el-icon><Plus /></el-icon>
          </button>
          <button class="action-btn" @click="showCreateGroup = true">
            <el-icon><Message /></el-icon>
          </button>
          <button class="action-btn" @click="showImportBots = true" title="导入聊天记录文件生成机器人">
            <el-icon><Download /></el-icon>
          </button>
          <button class="action-btn" @click="showQQImport = true" title="从QQ导入聊天记录生成机器人">
            <el-icon><ChatDotSquare /></el-icon>
          </button>
        </div>
      </div>
      <ContactList @select="onSelectContact" @group-info="onGroupInfo" @refresh="refreshContacts" />
    </div>

    <!-- Right Chat Area -->
    <div class="chat-area">
      <ChatWindow v-if="activeChat" :key="activeChat.key"
        :type="activeChat.type"
        :targetId="activeChat.id"
        :targetName="activeChat.name"
        @back="activeChat = null" />
      <div v-else class="no-chat">
        <div class="no-chat-icon">
          <el-icon :size="64" color="var(--primary-color)"><ChatDotRound /></el-icon>
        </div>
        <div class="no-chat-title">选择一个联系人开始聊天</div>
        <div class="no-chat-subtitle">您可以从左侧列表中选择好友或群聊</div>
      </div>
    </div>

    <!-- Add Friend Dialog -->
    <AddFriendDialog v-model:visible="showAddFriend" @done="onAddFriendDone" />

    <!-- Create Group Dialog -->
    <CreateGroupDialog v-model:visible="showCreateGroup" @done="onGroupCreated" />

    <!-- Friend Requests Dialog -->
    <el-dialog v-model="showFriendRequests" title="好友申请" width="450px">
      <div v-if="contactStore.pendingRequests.length === 0" class="empty-center">
        暂无待处理的好友申请
      </div>
      <div v-for="req in contactStore.pendingRequests" :key="req.id" class="request-item">
        <div class="request-info">
          <div class="request-avatar">{{ (req.nickname || req.username)[0] }}</div>
          <div>
            <div class="request-name">{{ req.nickname || req.username }}</div>
            <div class="request-username">@{{ req.username }}</div>
          </div>
        </div>
        <div class="request-actions">
          <el-button type="primary" size="small" @click="handleAccept(req.friendId)">接受</el-button>
          <el-button size="small" @click="handleReject(req.friendId)">拒绝</el-button>
        </div>
      </div>
    </el-dialog>

    <!-- Import Bots Dialog -->
    <ImportBotsDialog v-model:visible="showImportBots" @done="onBotsImported" />

    <!-- QQ Import Dialog -->
    <QQImportDialog v-model:visible="showQQImport" @done="onBotsImported" />

    <!-- Group Info Dialog -->
    <GroupInfoDialog v-model:visible="showGroupInfo" :group="selectedGroup" @refresh="refreshContacts" />

    <!-- Profile Dialog -->
    <ProfileDialog v-model:visible="showProfile" :display-name="userStore.nickname || userStore.username" @updated="onProfileUpdated" />

    <!-- Background Settings Dialog -->
    <BackgroundSettings v-model:visible="showBackground" />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '../store/user'
import { useContactStore } from '../store/contact'
import { useChatStore } from '../store/chat'
import { connectWebSocket, disconnectWebSocket, addMessageHandler, removeMessageHandler, addPresenceHandler, removePresenceHandler, subscribeGroupMessages } from '../utils/websocket'
import { acceptFriendRequest, rejectFriendRequest } from '../api/friend'
import { deleteAccount } from '../api/user'
import { ElMessage, ElMessageBox } from 'element-plus'
import { ArrowDown, UserFilled, Plus, Message, ChatDotRound, Download, ChatDotSquare } from '@element-plus/icons-vue'
import ContactList from '../components/ContactList.vue'
import ChatWindow from '../components/ChatWindow.vue'
import AddFriendDialog from '../components/AddFriendDialog.vue'
import CreateGroupDialog from '../components/CreateGroupDialog.vue'
import GroupInfoDialog from '../components/GroupInfoDialog.vue'
import ImportBotsDialog from '../components/ImportBotsDialog.vue'
import QQImportDialog from '../components/QQImportDialog.vue'
import ProfileDialog from '../components/ProfileDialog.vue'
import BackgroundSettings from '../components/BackgroundSettings.vue'

const router = useRouter()
const userStore = useUserStore()
const contactStore = useContactStore()
const chatStore = useChatStore()

const showAddFriend = ref(false)
const showCreateGroup = ref(false)
const showImportBots = ref(false)
const showQQImport = ref(false)
const showFriendRequests = ref(false)
const showGroupInfo = ref(false)
const showProfile = ref(false)
const showBackground = ref(false)
const selectedGroup = ref(null)
const activeChat = ref(null)

function onSelectContact(contact) {
  activeChat.value = contact
}

function onGroupInfo(group) {
  selectedGroup.value = { id: group.id, name: group.name }
  showGroupInfo.value = true
}

function openFriendRequests() {
  contactStore.fetchPendingRequests()
  showFriendRequests.value = true
}

async function onAddFriendDone() {
  showAddFriend.value = false
  await refreshContacts()
}

async function onGroupCreated() {
  showCreateGroup.value = false
  await refreshContacts()
}

async function onBotsImported() {
  showImportBots.value = false
  showQQImport.value = false
  await refreshContacts()
}

async function refreshContacts() {
  await contactStore.fetchAll()
}

function handleMessage(msg) {
  let key
  if (msg.messageType === 0) {
    const otherId = msg.senderId === userStore.userId ? msg.targetId : msg.senderId
    key = `private_${otherId}`
  } else {
    key = `group_${msg.targetId}`
  }
  chatStore.addMessage(key, {
    id: msg.messageId,
    messageType: msg.messageType,
    senderId: msg.senderId,
    senderName: msg.senderName,
    senderAvatar: msg.senderAvatar,
    targetId: msg.targetId,
    replyToId: msg.replyToId,
    replyToContent: msg.replyToContent,
    replyToSenderName: msg.replyToSenderName,
    content: msg.content,
    contentType: msg.contentType,
    createdAt: msg.createdAt
  })
}

function handlePresence(data) {
  contactStore.updateFriendStatus(data.userId, data.status === 'ONLINE')
}

async function handleAccept(friendId) {
  await acceptFriendRequest(friendId)
  ElMessage.success('已接受好友申请')
  await refreshContacts()
}

async function handleReject(friendId) {
  await rejectFriendRequest(friendId)
  ElMessage.success('已拒绝好友申请')
  await refreshContacts()
}

function handleUserCommand(command) {
  if (command === 'profile') {
    showProfile.value = true
  } else if (command === 'background') {
    showBackground.value = true
  } else if (command === 'deleteAccount') {
    confirmDeleteAccount()
  } else if (command === 'logout') {
    disconnectWebSocket()
    userStore.logout()
    router.push('/login')
  }
}

async function confirmDeleteAccount() {
  try {
    await ElMessageBox.confirm(
      '账户注销后将永久删除您的所有数据，包括好友关系、消息记录、群组成员资格。此操作不可撤销！',
      '确认注销账户',
      { confirmButtonText: '确认注销', cancelButtonText: '取消', type: 'warning' }
    )
    await deleteAccount()
    disconnectWebSocket()
    userStore.logout()
    ElMessage.success('账户已注销')
    router.push('/login')
  } catch (e) {
    if (e !== 'cancel') {
      // API error handled by interceptor
    }
  }
}

function onProfileUpdated() {
  ElMessage.success('个人资料已更新')
}

async function subscribeAllGroups() {
  const groups = contactStore.groupList
  for (const g of groups) {
    subscribeGroupMessages(g.id)
  }
}

onMounted(async () => {
  await userStore.fetchUser()
  await contactStore.fetchAll()

  const token = localStorage.getItem('token')
  if (token) {
    try {
      await connectWebSocket(token)
      addMessageHandler(handleMessage)
      addPresenceHandler(handlePresence)
      // Subscribe to all group topics
      subscribeAllGroups()
    } catch (e) {
      console.error('WebSocket connection failed:', e)
    }
  }
})

onUnmounted(() => {
  removeMessageHandler(handleMessage)
  removePresenceHandler(handlePresence)
  disconnectWebSocket()
})
</script>

<style scoped>
.chat-layout {
  display: flex;
  height: 100vh;
  background: var(--apple-gray-ultra-light);
}

.sidebar {
  width: 360px;
  background: var(--apple-white);
  display: flex;
  flex-direction: column;
  border-right: 1px solid rgba(0, 0, 0, 0.08);
  box-shadow: var(--apple-shadow-sm);
}

.sidebar-header {
  padding: 24px;
  background: var(--apple-gray-ultra-light);
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.user-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.user-avatar {
  width: 48px;
  height: 48px;
  border-radius: var(--apple-radius-md);
  background: linear-gradient(135deg, var(--apple-blue), #5856d6);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 500;
  color: white;
  overflow: hidden;
  cursor: pointer;
  transition: all var(--apple-transition-fast);
}

.user-avatar:hover {
  transform: scale(1.05);
}

.user-avatar .avatar-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.user-details {
  flex: 1;
  margin-left: 12px;
}

.nickname {
  font-size: 17px;
  font-weight: 600;
  margin-bottom: 3px;
  color: var(--apple-text-primary);
}

.status-indicator {
  display: flex;
  align-items: center;
  font-size: 12px;
  color: var(--apple-text-secondary);
}

.status-dot {
  width: 8px;
  height: 8px;
  background: var(--apple-success);
  border-radius: 50%;
  margin-right: 6px;
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.5;
    transform: scale(1.2);
  }
}

.dropdown-trigger {
  padding: 8px;
  border-radius: var(--apple-radius-sm);
  background: rgba(0, 0, 0, 0.04);
  transition: all var(--apple-transition-fast);
  cursor: pointer;
  color: var(--apple-text-secondary);
}

.dropdown-trigger:hover {
  background: rgba(0, 0, 0, 0.08);
  color: var(--apple-text-primary);
}

.header-actions {
  display: flex;
  gap: 8px;
}

.action-btn {
  width: 40px;
  height: 40px;
  border-radius: var(--apple-radius-md);
  background: rgba(0, 0, 0, 0.04);
  border: none;
  color: var(--apple-text-primary);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all var(--apple-transition-fast);
}

.action-btn:hover {
  background: rgba(0, 0, 0, 0.08);
  transform: scale(1.05);
}

.action-btn:active {
  transform: scale(0.95);
}

.chat-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: var(--apple-gray-light);
}

.no-chat {
  flex: 1;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 24px;
}

.no-chat-icon {
  width: 100px;
  height: 100px;
  border-radius: var(--apple-radius-xl);
  background: linear-gradient(135deg, rgba(0, 113, 227, 0.08), rgba(88, 86, 214, 0.08));
  display: flex;
  align-items: center;
  justify-content: center;
  animation: icon-float 3s ease-in-out infinite;
}

@keyframes icon-float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-8px);
  }
}

.no-chat-title {
  font-size: 20px;
  font-weight: 600;
  color: var(--apple-text-primary);
  margin-bottom: 6px;
}

.no-chat-subtitle {
  font-size: 15px;
  color: var(--apple-text-secondary);
}

.request-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 0;
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
  transition: background var(--apple-transition-fast);
}

.request-item:hover {
  background: var(--apple-gray-light);
}

.request-info {
  display: flex;
  align-items: center;
  gap: 12px;
}

.request-avatar {
  width: 44px;
  height: 44px;
  border-radius: var(--apple-radius-md);
  background: linear-gradient(135deg, var(--apple-blue), #5856d6);
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-weight: 500;
}

.request-name {
  font-weight: 500;
  color: var(--apple-text-primary);
}

.request-username {
  font-size: 13px;
  color: var(--apple-text-secondary);
}

.request-actions {
  display: flex;
  gap: 8px;
}

.empty-center {
  text-align: center;
  color: var(--apple-text-secondary);
  padding: 48px 24px;
}

:deep(.el-dropdown-menu__item) {
  color: var(--apple-text-primary);
}

:deep(.el-dropdown-menu__item:hover) {
  background: var(--apple-gray-light);
}
</style>
