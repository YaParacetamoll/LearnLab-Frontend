<script setup lang="ts">
    import { toast } from '@steveyuowo/vue-hot-toast'
    const userState = useUserState()

    const props = defineProps({
        p_id: { type: Number, required: true },
        c_id: { type: String, required: true },
    })

    const commentStore = ref()
    const comment = ref()
    const commentReplyTo = ref()
    const commentReplyToSub = ref()
    const visibility = ref(false)
    const isSending = ref(false)
    const isCommentLoading = ref(false)

    async function loadComments() {
        isCommentLoading.value = true
        const loadCommentToast = toast.loading('กำลังแสดงคอมเมนต์')
        await $fetch<{ status: number; message: string }>('/api/message/', {
            query: {
                p_receiver: props.p_id,
                c_id: props.c_id,
            },
        })
            .then((res) => {
                isCommentLoading.value = false
                commentStore.value = res
                toast.update(loadCommentToast, {
                    type: 'success',
                    message: res?.message,
                })
            })
            .catch((err) => {
                isCommentLoading.value = false
                toast.update(loadCommentToast, {
                    type: 'error',
                    message: err?.data?.message,
                })
            })
    }

    async function sendComment() {
        isSending.value = true
        const sendCommentToast = toast.loading('กำลังส่งคอมเมนต์')
        let payload = {
            p_receiver: props.p_id,
            c_id: props.c_id,
            m_content: comment.value,
        }
        if (commentReplyTo.value)
            Object.assign(payload, { m_thread: commentReplyTo.value })
        await $fetch<{ status: number; message: string }>('/api/message/', {
            method: 'PUT',
            body: payload,
        })
            .then((res) => {
                loadComments()
                comment.value = null
                isSending.value = false
                toast.update(sendCommentToast, {
                    type: 'success',
                    message: res?.message,
                })
            })
            .catch((err) => {
                isSending.value = false
                toast.update(sendCommentToast, {
                    type: 'error',
                    message: err?.data?.message,
                })
            })
    }

    async function deleteComment(m_id: number) {
        isSending.value = true
        const sendCommentToast = toast.loading('กำลังลบคอมเมนต์')
        let payload = {
            m_id: m_id,
        }
        await $fetch<{ status: number; message: string }>('/api/message/', {
            method: 'DELETE',
            body: payload,
        })
            .then((res) => {
                loadComments()
                isSending.value = false
                toast.update(sendCommentToast, {
                    type: 'success',
                    message: res?.message,
                })
            })
            .catch((err) => {
                isSending.value = false
                toast.update(sendCommentToast, {
                    type: 'error',
                    message: err?.data?.message,
                })
            })
    }
</script>
<template>
    <span
        v-if="!commentStore"
        class="cursor-pointer underline"
        @click="
            () => {
                loadComments()
                visibility = true
            }
        ">
        Load Comments
    </span>
    <span
        v-else-if="!visibility && commentStore"
        class="cursor-pointer underline"
        @click="
            () => {
                visibility = true
            }
        ">
        Load Comments
    </span>
    <span v-else class="cursor-pointer underline" @click="visibility = false">
        Hide Comments
    </span>
    <div class="flex flex-row gap-2">
        <div class="relative flex-grow">
            <input
                type="text"
                v-model="comment"
                id="hs-floating-comment-crs"
                placeholder="หัวข้อโพสต์"
                class="peer block w-full rounded-lg border-gray-200 p-4 text-sm placeholder:text-transparent autofill:pb-2 autofill:pt-6 focus:border-blue-500 focus:pb-2 focus:pt-6 focus:ring-blue-500 disabled:pointer-events-none disabled:opacity-50 [&:not(:placeholder-shown)]:pb-2 [&:not(:placeholder-shown)]:pt-6" />
            <label
                for="hs-floating-comment-crs"
                class="pointer-events-none absolute start-0 top-0 h-full truncate border border-transparent p-4 text-sm transition duration-100 ease-in-out peer-focus:-translate-y-1.5 peer-focus:text-xs peer-focus:text-gray-500 peer-disabled:pointer-events-none peer-disabled:opacity-50 peer-[:not(:placeholder-shown)]:-translate-y-1.5 peer-[:not(:placeholder-shown)]:text-xs peer-[:not(:placeholder-shown)]:text-gray-500">
                {{
                    commentReplyTo
                        ? `คอมเมนต์ (กำลังตอบกลับ ${commentReplyTo})`
                        : 'คอมเมนต์'
                }}
            </label>
        </div>
        <button
            type="button"
            :disabled="!comment || isSending || isCommentLoading"
            @click="sendComment()"
            class="transition-color inline-flex w-16 items-center justify-center gap-x-2 rounded-lg border border-transparent bg-blue-600 px-3 py-2 text-sm font-semibold text-white duration-200 ease-in-out hover:bg-blue-700 disabled:pointer-events-none disabled:opacity-50">
            <span class="material-icons-outlined no-underline">send</span>
        </button>
    </div>
    <div
        v-if="!isCommentLoading"
        v-show="visibility"
        v-for="comment in commentStore"
        class="flex flex-col gap-2">
        <div
            class="transition-color border-1 ml-4 flex flex-col gap-1 rounded-md border px-4 py-2 duration-200 ease-in-out"
            :class="
                commentReplyTo === comment.m_id
                    ? 'border-2 border-blue-600'
                    : 'border-1'
            ">
            <div class="font-bold">
                {{ comment.u_firstname }} {{ comment.u_lastname }}
                <span class="text-sm font-light">
                    {{ new Date(comment.created_at).toLocaleString() }}
                </span>
            </div>
            <div>{{ comment.m_content }}</div>
            <div class="flex flex-row">
                <div
                    class="inline-flex cursor-pointer items-center text-xs hover:underline"
                    @click="
                        () => {
                            if (commentReplyTo == comment.m_id) {
                                commentReplyTo = null
                            } else {
                                commentReplyTo = comment.m_id
                                commentReplyToSub = null
                            }
                        }
                    ">
                    <span
                        class="material-icons-outlined no-underline"
                        style="font-size: 20px">
                        reply
                    </span>
                    ตอบกลับ
                </div>
                <div
                    v-if="comment.m_sender === userState?.u_id"
                    @click="deleteComment(comment.m_id)"
                    class="inline-flex cursor-pointer items-center text-xs hover:underline">
                    <span
                        class="material-icons-outlined no-underline"
                        style="font-size: 20px">
                        delete
                    </span>
                    ลบ
                </div>
            </div>
        </div>
        <div
            v-for="replies in comment.replies"
            class="transition-color border-1 ml-12 flex flex-col gap-1 rounded-md border px-4 py-2 duration-200 ease-in-out"
            :class="
                commentReplyToSub === replies.m_id
                    ? 'border-2 border-blue-600'
                    : 'border-1'
            ">
            <div class="font-bold">
                {{ replies.u_firstname }} {{ replies.u_lastname }}
                <span class="text-sm font-light">
                    {{ new Date(comment.created_at).toLocaleString() }}
                </span>
            </div>
            <div>{{ replies.m_content }}</div>
            <div class="flex flex-row">
                <div
                    class="inline-flex cursor-pointer items-center text-xs hover:underline"
                    @click="
                        () => {
                            if (commentReplyToSub == replies.m_id) {
                                commentReplyTo = null
                                commentReplyToSub = null
                            } else {
                                commentReplyToSub = replies.m_id
                                commentReplyTo = replies.m_thread
                            }
                        }
                    ">
                    <span
                        class="material-icons-outlined no-underline"
                        style="font-size: 20px">
                        reply
                    </span>
                    ตอบกลับ
                </div>
                <div
                    v-if="replies.m_sender === userState?.u_id"
                    @click="deleteComment(replies.m_id)"
                    class="inline-flex cursor-pointer items-center text-xs hover:underline">
                    <span
                        class="material-icons-outlined no-underline"
                        style="font-size: 20px">
                        delete
                    </span>
                    ลบ
                </div>
            </div>
        </div>
    </div>
    <div
        v-else
        class="border-1 flex w-full flex-row gap-2 rounded-md border p-4">
        <div
            class="inline-block size-6 animate-spin rounded-full border-[3px] border-current border-t-transparent text-blue-600 dark:text-blue-500"
            role="status"
            aria-label="loading">
            <span class="sr-only">Loading...</span>
        </div>
        กำลังโหลดข้อมูล
    </div>
</template>
