<script setup lang="ts">
import route from 'ziggy-js';
import { Link, router } from '@inertiajs/vue3';
import { ref } from 'vue';
import {
  ApplicationMark,
  DropdownInput,
  DropdownLink,
  NavLink,
  ResponsiveNavLink,
} from '~~/Components';

const showingNavigationDropdown = ref(false);

const switchToTeam = (team: any) => {
  router.put(
    route('current-team.update'),
    {
      team_id: team.id,
    },
    {
      preserveState: false,
    }
  );
};

const logout = () => {
  router.post(route('logout'));
};
</script>

<template>
  <nav
    class="bg-white dark:bg-gray-500 border-b border-gray-100 dark:border-gray-400"
  >
    <!-- Primary Navigation Menu -->
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex justify-between h-16">
        <div class="flex">
          <!-- Logo -->
          <div class="shrink-0 flex items-center">
            <Link :href="route('dashboard')">
              <ApplicationMark class="block h-9 w-auto" />
            </Link>
          </div>

          <!-- Navigation Links -->
          <div class="hidden space-x-8 sm:-my-px sm:ml-10 sm:flex">
            <NavLink
              :href="route('dashboard')"
              :active="route().current('dashboard')"
            >
              Dashboard
            </NavLink>
            <NavLink
              :href="route('library.index')"
              :active="route().current('library.index')"
            >
              Library
            </NavLink>
            <NavLink
              :href="route('shared.index')"
              :active="route().current('shared.index')"
            >
              Shared
            </NavLink>
          </div>
        </div>

        <div class="hidden sm:flex sm:items-center sm:ml-6">
          <div class="ml-3 relative">
            <Link
              :href="route('library.create')"
              class="inline-flex items-center px-4 py-2 bg-gray-800 border border-transparent rounded-md font-semibold text-xs text-white uppercase tracking-widest hover:bg-gray-700 active:bg-gray-900 focus:outline-none focus:border-gray-900 focus:ring focus:ring-gray-300 disabled:opacity-25 transition"
            >
              New Lick
            </Link>
          </div>
          <div class="ml-3 relative">
            <!-- Teams Dropdown -->
            <DropdownInput
              v-if="$page.props.jetstream.hasTeamFeatures"
              align="right"
              width="60"
              :content-classes="['py-1', 'bg-white', 'dark:bg-gray-500']"
            >
              <template #trigger>
                <span class="inline-flex rounded-md">
                  <button
                    type="button"
                    class="inline-flex items-center px-3 py-2 border border-transparent leading-4 font-medium rounded-md text-gray-500 bg-white hover:bg-gray-50 hover:text-gray-700 dark:text-gray-300 dark:bg-gray-500 dark:hover:bg-gray-600 dark:hover:text-gray-200 focus:outline-none focus:bg-gray-50 active:bg-gray-50 dark:focus:bg-gray-800 dark:active:bg-gray-800 transition"
                  >
                    {{ $page.props.auth.user.current_team?.name }}

                    <svg
                      class="ml-2 -mr-0.5 h-4 w-4"
                      xmlns="http://www.w3.org/2000/svg"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke-width="1.5"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        d="M8.25 15L12 18.75 15.75 15m-7.5-6L12 5.25 15.75 9"
                      />
                    </svg>
                  </button>
                </span>
              </template>

              <template #content>
                <div class="w-60">
                  <!-- Team Management -->
                  <template v-if="$page.props.jetstream.hasTeamFeatures">
                    <div class="block px-4 py-2 text-sm text-gray-400">
                      Manage Team
                    </div>

                    <!-- Team Settings -->
                    <DropdownLink
                      :href="
                        route('teams.show', $page.props.auth.user.current_team)
                      "
                    >
                      Team Settings
                    </DropdownLink>

                    <DropdownLink
                      v-if="$page.props.jetstream.canCreateTeams"
                      :href="route('teams.create')"
                    >
                      Create New Team
                    </DropdownLink>

                    <div
                      class="border-t border-gray-100 dark:border-gray-400"
                    />

                    <!-- Team Switcher -->
                    <div class="block px-4 py-2 text-sm text-gray-400">
                      Switch Teams
                    </div>

                    <template
                      v-for="team in $page.props.auth.user.all_teams"
                      :key="team.id"
                    >
                      <form @submit.prevent="switchToTeam(team)">
                        <DropdownLink as="button">
                          <div class="flex items-center">
                            <svg
                              v-if="
                                team.id == $page.props.auth.user.current_team_id
                              "
                              class="mr-2 h-5 w-5 text-green-400"
                              xmlns="http://www.w3.org/2000/svg"
                              fill="none"
                              viewBox="0 0 24 24"
                              stroke-width="1.5"
                              stroke="currentColor"
                            >
                              <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                              />
                            </svg>

                            <div>{{ team.name }}</div>
                          </div>
                        </DropdownLink>
                      </form>
                    </template>
                  </template>
                </div>
              </template>
            </DropdownInput>
          </div>

          <!-- Settings Dropdown -->
          <div class="ml-3 relative">
            <DropdownInput
              align="right"
              width="48"
              :content-classes="['py-1', 'bg-white', 'dark:bg-gray-500']"
            >
              <template #trigger>
                <button
                  v-if="$page.props.jetstream.managesProfilePhotos"
                  class="flex border-2 border-transparent rounded-full focus:outline-none focus:border-gray-300 transition"
                >
                  <img
                    class="h-8 w-8 rounded-full object-cover"
                    :src="$page.props.auth.user.profile_photo_url"
                    :alt="$page.props.auth.user.name"
                  />
                </button>

                <span v-else class="inline-flex rounded-md">
                  <button
                    type="button"
                    class="inline-flex items-center px-3 py-2 border border-transparent leading-4 font-medium rounded-md text-gray-500 bg-white hover:bg-gray-50 hover:text-gray-700 dark:text-gray-300 dark:bg-gray-500 dark:hover:bg-gray-600 dark:hover:text-gray-200 focus:outline-none focus:bg-gray-50 active:bg-gray-50 dark:focus:bg-gray-800 dark:active:bg-gray-800 transition"
                  >
                    {{ $page.props.auth.user.name }}

                    <svg
                      class="ml-2 -mr-0.5 h-4 w-4"
                      xmlns="http://www.w3.org/2000/svg"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke-width="1.5"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        d="M4.5 12.75l6 6 9-13.5"
                      />
                    </svg>
                  </button>
                </span>
              </template>

              <template #content>
                <!-- Account Management -->
                <div class="block px-4 py-2 text-sm text-gray-400">
                  Manage Account
                </div>

                <DropdownLink :href="route('profile.show')">
                  Profile
                </DropdownLink>

                <DropdownLink
                  v-if="$page.props.jetstream.hasApiFeatures"
                  :href="route('api-tokens.index')"
                >
                  API Tokens
                </DropdownLink>

                <div class="border-t border-gray-100 dark:border-gray-400" />

                <!-- Authentication -->
                <form @submit.prevent="logout">
                  <DropdownLink as="button"> Log Out </DropdownLink>
                </form>
              </template>
            </DropdownInput>
          </div>
        </div>

        <!-- Hamburger -->
        <div class="-mr-2 flex items-center sm:hidden">
          <button
            class="inline-flex items-center justify-center p-2 rounded-md text-gray-400 hover:text-gray-500 hover:bg-gray-100 focus:outline-none focus:bg-gray-100 focus:text-gray-500 transition"
            @click="showingNavigationDropdown = !showingNavigationDropdown"
          >
            <svg
              class="h-6 w-6"
              stroke="currentColor"
              fill="none"
              viewBox="0 0 24 24"
            >
              <path
                :class="{
                  hidden: showingNavigationDropdown,
                  'inline-flex': !showingNavigationDropdown,
                }"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M4 6h16M4 12h16M4 18h16"
              />
              <path
                :class="{
                  hidden: !showingNavigationDropdown,
                  'inline-flex': showingNavigationDropdown,
                }"
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M6 18L18 6M6 6l12 12"
              />
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Responsive Navigation Menu -->
    <div
      :class="{
        block: showingNavigationDropdown,
        hidden: !showingNavigationDropdown,
      }"
      class="sm:hidden"
    >
      <div class="pt-2 pb-3 space-y-1">
        <ResponsiveNavLink
          :href="route('dashboard')"
          :active="route().current('dashboard')"
        >
          Dashboard
        </ResponsiveNavLink>
        <ResponsiveNavLink
          :href="route('library.index')"
          :active="route().current('library.index')"
        >
          Library
        </ResponsiveNavLink>
        <ResponsiveNavLink
          :href="route('shared.index')"
          :active="route().current('shared.index')"
        >
          Shared
        </ResponsiveNavLink>
      </div>

      <!-- Responsive Settings Options -->
      <div class="pt-4 pb-1 border-t border-gray-200">
        <div class="flex items-center px-4">
          <div
            v-if="$page.props.jetstream.managesProfilePhotos"
            class="shrink-0 mr-3"
          >
            <img
              class="h-10 w-10 rounded-full object-cover"
              :src="$page.props.auth.user.profile_photo_url"
              :alt="$page.props.auth.user.name"
            />
          </div>

          <div>
            <div class="font-medium text-base text-gray-800">
              {{ $page.props.auth.user.name }}
            </div>
            <div class="font-medium text-gray-500">
              {{ $page.props.auth.user.email }}
            </div>
          </div>
        </div>

        <div class="mt-3 space-y-1">
          <ResponsiveNavLink
            :href="route('profile.show')"
            :active="route().current('profile.show')"
          >
            Profile
          </ResponsiveNavLink>

          <ResponsiveNavLink
            v-if="$page.props.jetstream.hasApiFeatures"
            :href="route('api-tokens.index')"
            :active="route().current('api-tokens.index')"
          >
            API Tokens
          </ResponsiveNavLink>

          <!-- Authentication -->
          <form method="POST" @submit.prevent="logout">
            <ResponsiveNavLink as="button"> Log Out </ResponsiveNavLink>
          </form>

          <!-- Team Management -->
          <template v-if="$page.props.jetstream.hasTeamFeatures">
            <div class="border-t border-gray-200" />

            <div class="block px-4 py-2 text-sm text-gray-400">Manage Team</div>

            <!-- Team Settings -->
            <ResponsiveNavLink
              :href="route('teams.show', $page.props.auth.user.current_team)"
              :active="route().current('teams.show')"
            >
              Team Settings
            </ResponsiveNavLink>

            <ResponsiveNavLink
              v-if="$page.props.jetstream.canCreateTeams"
              :href="route('teams.create')"
              :active="route().current('teams.create')"
            >
              Create New Team
            </ResponsiveNavLink>

            <div class="border-t border-gray-200" />

            <!-- Team Switcher -->
            <div class="block px-4 py-2 text-sm text-gray-400">
              Switch Teams
            </div>

            <template
              v-for="team in $page.props.auth.user.all_teams"
              :key="team.id"
            >
              <form @submit.prevent="switchToTeam(team)">
                <ResponsiveNavLink as="button">
                  <div class="flex items-center">
                    <svg
                      v-if="team.id == $page.props.auth.user.current_team_id"
                      class="mr-2 h-5 w-5 text-green-400"
                      xmlns="http://www.w3.org/2000/svg"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke-width="1.5"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        d="M9 12.75L11.25 15 15 9.75M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                      />
                    </svg>
                    <div>{{ team.name }}</div>
                  </div>
                </ResponsiveNavLink>
              </form>
            </template>
          </template>
        </div>
      </div>
    </div>
  </nav>
</template>
