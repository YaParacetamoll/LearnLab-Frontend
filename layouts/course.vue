<script setup lang="ts">
    import { toast } from '@steveyuowo/vue-hot-toast'

    useSeoMeta({
        title: 'LearnLab: Course Page',
        description: 'Course page',
    })
    const route = useRoute()
    const userRole = useUserCourseState()
    const crs_info = ref()
    const crs_pending = ref(true)
    const crs_member = ref()
    let sessionInfo = ref()
    watch(
        () => route.query.id,
        async (id) => {
            fetchCourse(id)
            fetchMember(id)
        }
    )

    async function fetchSessionID() {
        await $fetch('/api/session/', {
            credentials: 'include',
        }).then((res) => {
            sessionInfo.value = res
        })
    }

    async function destroySession() {
        await $fetch('/api/session/', {
            method: 'DELETE',
            credentials: 'include',
        }).then((res) => {
            sessionInfo.value = res
        })
    }

    async function fetchCourse(id: number) {
        crs_pending.value = true
        await $fetch<CoursePageAPIPUTResponse>('/api/courses/', {
            method: 'PUT',
            body: { c_id: id },
        })
            .then((res) => {
                crs_info.value = res
                crs_pending.value = false
            })
            .catch((err) => {
                toast.error(err?.data?.message)
                navigateTo('/mycourse', { replace: true })
            })
    }

    async function fetchMember(id: number) {
        crs_pending.value = true
        await $fetch('/api/courses/enroll/', {
            query: { c_id: id, u_role: 'INSTRUCTOR' },
        })
            .then((res) => {
                crs_member.value = res
                crs_pending.value = false
            })
            .catch((err) => {
                toast.error(err?.data?.message)
                navigateTo('/mycourse', { replace: true })
            })
    }

    async function fetchUserRoleState() {
        await $fetch<UserRoles>('/api/courses/me/?my_course_role', {})
            .then((res) => {
                userRole.value = res
            })
            .catch((err) => {})
    }
    const bannerImage = ref()
    const handleBrokenImage = () => {
        bannerImage.value.src = '/images/CourseBannerDefault.svg'
    }

    if (route.query.id) {
        fetchCourse(route.query.id)
        fetchMember(route.query.id)
    }
    fetchUserRoleState()
</script>
<template>
    <NavigationBar />
    <AccountModal />
    <div class="mx-auto mt-20 max-w-screen-2xl">
        <div class="flex flex-col gap-4 px-4 pb-16">
            <div class="relative flex gap-x-4">
                <NuxtLink
                    to="/mycourse"
                    class="inline-flex items-center gap-x-2 rounded-lg border border-transparent px-3 py-2 text-sm font-semibold text-blue-600 transition-all duration-200 ease-in-out hover:bg-blue-100 hover:text-blue-800 disabled:pointer-events-none disabled:opacity-50">
                    <span class="material-icons-outlined">home</span>
                    กลับหน้าคอร์ส
                </NuxtLink>
                <div class="flex items-center">
                    Courses
                    <span class="material-icons-outlined">chevron_right</span>
                    View
                    <span class="material-icons-outlined">chevron_right</span>
                    {{ route.query.id }}
                </div>
            </div>
            <div class="relative flex">
                <img
                    class="h-full max-h-96 min-h-56 w-full rounded-xl object-cover object-[0%_50%]"
                    loading="lazy"
                    ref="bannerImage"
                    :src="
                        crs_info?.c_banner
                            ? `/api/courses/banner/?c_id=${crs_info?.c_id}`
                            : '/images/CourseBannerDefault.svg'
                    "
                    alt="Image Description"
                    @error="handleBrokenImage" />
                <div
                    class="absolute h-full w-full rounded-xl bg-gradient-to-b from-slate-50/0 from-50% to-zinc-900 sm:from-70%"></div>
                <div
                    v-if="userRole?.[route.query.id] === 'INSTRUCTOR'"
                    class="absolute right-0">
                    <div class="flex flex-row pr-2 pt-2">
                        <NuxtLink
                            :to="`/courses/edit?id=${route.query.id}`"
                            class="transition-color inline-flex items-center gap-x-2 rounded-lg px-3 py-2 text-sm font-semibold text-white drop-shadow-[0_0_8px_rgba(0,0,0,1)] duration-200 ease-in-out hover:text-blue-300 disabled:pointer-events-none disabled:opacity-50">
                            <span class="material-icons-outlined">edit</span>
                            แก้ไข
                        </NuxtLink>
                    </div>
                </div>
                <div
                    v-if="userRole?.[route.query.id] === 'INSTRUCTOR'"
                    class="absolute left-0">
                    <div
                        class="flex flex-col items-center justify-center pl-4 pt-4 text-white drop-shadow-[0_0_8px_rgba(0,0,0,1)]">
                        <span class="font-md text-xl">รหัสเข้าคอร์ส</span>
                        <span class="font-mono text-2xl font-light">
                            {{ crs_info?.c_code }}
                        </span>
                    </div>
                </div>
                <div
                    v-if="!crs_pending"
                    class="absolute bottom-5 left-5 flex flex-col gap-2">
                    <span
                        class="line-clamp-1 text-4xl font-bold text-white drop-shadow-[0_0_12px_rgba(0,0,0,1)]">
                        {{ crs_info?.c_name }}
                    </span>
                    <span
                        class="bottom-0 h-auto max-h-10 overflow-auto text-sm font-light text-white drop-shadow-[0_0_12px_rgba(0,0,0,1)]">
                        {{ crs_info?.c_description }}
                    </span>
                </div>
                <div
                    v-else
                    class="absolute bottom-5 left-5 flex w-full animate-pulse flex-col gap-2">
                    <span class="text-4xl font-bold text-white">
                        <h3 class="h-8 w-[200px] rounded-full bg-white"></h3>
                    </span>
                    <span
                        class="bottom-0 h-3 max-h-10 w-full rounded-full bg-white text-sm font-light text-white"
                        style="width: 90%"></span>
                </div>
            </div>
            <div class="flex flex-col gap-4 md:flex-row">
                <nav
                    class="border-1 flex h-fit w-full flex-col rounded-lg border shadow-sm md:w-64">
                    <NuxtLink
                        :to="`/courses/view?id=${route.query.id}`"
                        :class="
                            route.path === '/courses/view'
                                ? 'bg-blue-600 text-white hover:bg-blue-700'
                                : 'text-black hover:bg-blue-300/50'
                        "
                        class="nav-menu">
                        <span class="material-icons-outlined">home</span>
                        หน้าหลัก
                    </NuxtLink>
                    <NuxtLink
                        :to="`/courses/material?id=${route.query.id}`"
                        :class="
                            route.path === '/courses/material'
                                ? 'bg-blue-600 text-white hover:bg-blue-700'
                                : 'text-black hover:bg-blue-300/50'
                        "
                        class="nav-menu">
                        <span class="material-icons-outlined">menu_book</span>
                        เนื้อหาการสอน
                    </NuxtLink>
                    <NuxtLink
                        :to="`/courses/assignment?id=${route.query.id}`"
                        :class="
                            route.path === '/courses/assignment'
                                ? 'bg-blue-600 text-white hover:bg-blue-700'
                                : 'text-black hover:bg-blue-300/50'
                        "
                        class="nav-menu">
                        <span class="material-icons-outlined">edit_note</span>
                        งานมอบหมาย
                    </NuxtLink>
                    <NuxtLink
                        :to="`/courses/quiz?id=${route.query.id}`"
                        :class="
                            route.path === '/courses/quiz'
                                ? 'bg-blue-600 text-white hover:bg-blue-700'
                                : 'text-black hover:bg-blue-300/50'
                        "
                        class="nav-menu">
                        <span class="material-icons-outlined">quiz</span>
                        แบบทดสอบ
                    </NuxtLink>
                    <hr />
                    <div class="flex items-center gap-2 p-2 text-lg font-bold">
                        <span class="material-icons-outlined">people</span>
                        ผู้สอน
                    </div>
                    <div
                        class="flex flex-row items-center gap-2 p-2"
                        v-for="inst in crs_member">
                        <div v-if="inst.u_avatar" class="h-8 w-8 rounded-md">
                            <img
                                class="bottom-1 aspect-square rounded-md border object-cover"
                                :src="`/api/avatar/?u_id=${inst.u_id}`" />
                        </div>
                        <div
                            class="flex h-8 w-8 select-none flex-col items-center justify-center rounded-md bg-slate-200 text-xl"
                            v-if="!inst?.u_avatar">
                            {{
                                `${inst?.u_firstname.slice(0, 1)}${inst?.u_lastname.slice(0, 1)}`
                            }}
                        </div>
                        <div class="flex flex-col">
                            <span>
                                {{ inst?.u_firstname }} {{ inst?.u_lastname }}
                            </span>
                            <span class="cursor-pointer text-xs text-slate-400">
                                <NuxtLink :to="`mailto:${inst?.u_email}`">
                                    {{ inst?.u_email }}
                                </NuxtLink>
                            </span>
                        </div>
                    </div>
                </nav>
                <main class="flex w-full flex-col gap-y-4">
                    <slot />
                </main>
            </div>
        </div>
    </div>
    <Footer />
</template>

<style scoped>
    .nav-menu {
        @apply flex h-12 cursor-pointer items-center gap-4 rounded-lg p-4 transition-colors duration-200 ease-in-out;
    }

    ::-webkit-scrollbar {
        width: 20px;
    }

    ::-webkit-scrollbar-track {
        background-color: transparent;
    }

    ::-webkit-scrollbar-thumb {
        background-color: #d6dee1;
        border-radius: 20px;
        border: 6px solid transparent;
        background-clip: content-box;
    }

    ::-webkit-scrollbar-thumb:hover {
        background-color: #a8bbbf;
    }
</style>
