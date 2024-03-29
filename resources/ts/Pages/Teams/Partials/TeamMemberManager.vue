<script setup lang="ts">
import { PropType, ref } from 'vue';
import { router, useForm, usePage } from '@inertiajs/vue3';
import {
  ActionMessage,
  ActionSection,
  ConfirmationModal,
  DangerButton,
  DialogModal,
  FormSection,
  InputError,
  InputLabel,
  PrimaryButton,
  SecondaryButton,
  SectionBorder,
  TextInput,
} from '~~/Components';
import route from 'ziggy-js';
import { Role, Team, TeamInvitation, User, UserPermissions } from '~~/types';

const props = defineProps({
  team: {
    type: Object as PropType<Team>,
    required: true,
  },
  availableRoles: {
    type: Array as PropType<Role[]>,
    default: () => [],
  },
  userPermissions: {
    type: Object as PropType<UserPermissions>,
    required: true,
  },
});

const addTeamMemberForm = useForm<{
  email: string;
  role: string | null;
}>({
  email: '',
  role: null,
});

const updateRoleForm = useForm<{ role: string | null }>({
  role: null,
});

const leaveTeamForm = useForm({});
const removeTeamMemberForm = useForm({});

const currentlyManagingRole = ref(false);
const managingRoleFor = ref<User | null>(null);
const confirmingLeavingTeam = ref(false);
const teamMemberBeingRemoved = ref<User | null>(null);

const addTeamMember = () => {
  addTeamMemberForm.post(route('team-members.store', props.team), {
    errorBag: 'addTeamMember',
    preserveScroll: true,
    onSuccess: () => addTeamMemberForm.reset(),
  });
};

const cancelTeamInvitation = (invitation: TeamInvitation) => {
  router.delete(route('team-invitations.destroy', invitation), {
    preserveScroll: true,
  });
};

const manageRole = (teamMember: User) => {
  managingRoleFor.value = teamMember;
  updateRoleForm.role = teamMember.membership.role;
  currentlyManagingRole.value = true;
};

const updateRole = () => {
  updateRoleForm.put(
    route('team-members.update', [
      props.team.id,
      managingRoleFor.value?.id ?? '',
    ]),
    {
      preserveScroll: true,
      onSuccess: () => (currentlyManagingRole.value = false),
    }
  );
};

const confirmLeavingTeam = () => {
  confirmingLeavingTeam.value = true;
};

const leaveTeam = () => {
  leaveTeamForm.delete(
    route('team-members.destroy', [props.team.id, usePage().props.auth.user.id])
  );
};

const confirmTeamMemberRemoval = (teamMember: User) => {
  teamMemberBeingRemoved.value = teamMember;
};

const removeTeamMember = () => {
  removeTeamMemberForm.delete(
    route('team-members.destroy', [
      props.team.id,
      teamMemberBeingRemoved.value?.id ?? '',
    ]),
    {
      errorBag: 'removeTeamMember',
      preserveScroll: true,
      preserveState: true,
      onSuccess: () => (teamMemberBeingRemoved.value = null),
    }
  );
};

const displayableRole = (role: string) => {
  return props.availableRoles.find((r) => r.key === role)?.name;
};
</script>

<template>
  <div>
    <div v-if="userPermissions.canAddTeamMembers">
      <SectionBorder />

      <!-- Add Team Member -->
      <FormSection @submitted="addTeamMember">
        <template #title> Add Team Member </template>

        <template #description>
          Add a new team member to your team, allowing them to collaborate with
          you.
        </template>

        <template #form>
          <div class="col-span-6">
            <div class="max-w-xl text-sm text-gray-600 dark:text-gray-300">
              Please provide the email address of the person you would like to
              add to this team.
            </div>
          </div>

          <!-- Member Email -->
          <div class="col-span-6 sm:col-span-4">
            <InputLabel for="email" value="Email" />
            <TextInput
              id="email"
              v-model="addTeamMemberForm.email"
              type="email"
              class="mt-1 block w-full"
            />
            <InputError
              :message="addTeamMemberForm.errors.email"
              class="mt-2"
            />
          </div>

          <!-- Role -->
          <div
            v-if="availableRoles.length > 0"
            class="col-span-6 lg:col-span-4"
          >
            <InputLabel for="roles" value="Role" />
            <InputError :message="addTeamMemberForm.errors.role" class="mt-2" />

            <div
              class="relative z-0 mt-1 border border-gray-200 dark:border-gray-400 rounded-lg cursor-pointer"
            >
              <button
                v-for="(role, i) in availableRoles"
                :key="role.key"
                type="button"
                class="relative px-4 py-3 inline-flex w-full rounded-lg focus:z-10 focus:outline-none focus:border-blue-300 focus:ring focus:ring-blue-200"
                :class="{
                  'border-t border-gray-200 dark:border-gray-400 rounded-t-none':
                    i > 0,
                  'rounded-b-none': i != Object.keys(availableRoles).length - 1,
                }"
                @click="addTeamMemberForm.role = role.key"
              >
                <div
                  :class="{
                    'opacity-50':
                      addTeamMemberForm.role &&
                      addTeamMemberForm.role != role.key,
                  }"
                >
                  <!-- Role Name -->
                  <div class="flex items-center">
                    <div
                      class="text-sm text-gray-600 dark:text-gray-300"
                      :class="{
                        'font-semibold': addTeamMemberForm.role == role.key,
                      }"
                    >
                      {{ role.name }}
                    </div>

                    <svg
                      v-if="addTeamMemberForm.role == role.key"
                      class="ml-2 h-5 w-5 text-green-400"
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
                  </div>

                  <!-- Role Description -->
                  <div
                    class="mt-2 text-xs text-gray-600 dark:text-gray-300 text-left"
                  >
                    {{ role.description }}
                  </div>
                </div>
              </button>
            </div>
          </div>
        </template>

        <template #actions>
          <ActionMessage
            :on="addTeamMemberForm.recentlySuccessful"
            class="mr-3"
          >
            Added.
          </ActionMessage>

          <PrimaryButton
            :class="{ 'opacity-25': addTeamMemberForm.processing }"
            :disabled="addTeamMemberForm.processing"
          >
            Add
          </PrimaryButton>
        </template>
      </FormSection>
    </div>

    <div
      v-if="
        team.team_invitations.length > 0 && userPermissions.canAddTeamMembers
      "
    >
      <SectionBorder />

      <!-- Team Member Invitations -->
      <ActionSection class="mt-10 sm:mt-0">
        <template #title> Pending Team Invitations </template>

        <template #description>
          These people have been invited to your team and have been sent an
          invitation email. They may join the team by accepting the email
          invitation.
        </template>

        <!-- Pending Team Member Invitation List -->
        <template #content>
          <div class="space-y-6">
            <div
              v-for="invitation in team.team_invitations"
              :key="invitation.id"
              class="flex items-center justify-between"
            >
              <div class="text-gray-600 dark:text-gray-300">
                {{ invitation.email }}
              </div>

              <div class="flex items-center">
                <!-- Cancel Team Invitation -->
                <button
                  v-if="userPermissions.canRemoveTeamMembers"
                  class="cursor-pointer ml-6 text-sm text-red-500 dark:text-red-800 focus:outline-none"
                  @click="cancelTeamInvitation(invitation)"
                >
                  Cancel
                </button>
              </div>
            </div>
          </div>
        </template>
      </ActionSection>
    </div>

    <div v-if="team.users.length > 0">
      <SectionBorder />

      <!-- Manage Team Members -->
      <ActionSection class="mt-10 sm:mt-0">
        <template #title> Team Members </template>

        <template #description>
          All of the people that are part of this team.
        </template>

        <!-- Team Member List -->
        <template #content>
          <div class="space-y-6">
            <div
              v-for="user in team.users"
              :key="user.id"
              class="flex items-center justify-between"
            >
              <div class="flex items-center">
                <img
                  class="w-8 h-8 rounded-full"
                  :src="user.profile_photo_url"
                  :alt="user.name"
                />
                <div class="ml-4 dark:text-gray-400">
                  {{ user.name }}
                </div>
              </div>

              <div class="flex items-center">
                <!-- Manage Team Member Role -->
                <button
                  v-if="
                    userPermissions.canAddTeamMembers && availableRoles.length
                  "
                  class="ml-2 text-sm text-gray-400 underline"
                  @click="manageRole(user)"
                >
                  {{ displayableRole(user.membership.role) }}
                </button>

                <div
                  v-else-if="availableRoles.length"
                  class="ml-2 text-sm text-gray-400"
                >
                  {{ displayableRole(user.membership.role) }}
                </div>

                <!-- Leave Team -->
                <button
                  v-if="$page.props.auth.user.id === user.id"
                  class="cursor-pointer ml-6 text-sm text-red-500 dark:text-red-800"
                  @click="confirmLeavingTeam"
                >
                  Leave
                </button>

                <!-- Remove Team Member -->
                <button
                  v-else-if="userPermissions.canRemoveTeamMembers"
                  class="cursor-pointer ml-6 text-sm text-red-500 dark:text-red-800"
                  @click="confirmTeamMemberRemoval(user)"
                >
                  Remove
                </button>
              </div>
            </div>
          </div>
        </template>
      </ActionSection>
    </div>

    <!-- Role Management Modal -->
    <DialogModal
      :show="currentlyManagingRole"
      @close="currentlyManagingRole = false"
    >
      <template #title> Manage Role </template>

      <template #content>
        <div v-if="managingRoleFor">
          <div
            class="relative z-0 mt-1 border border-gray-200 dark:border-gray-400 rounded-lg cursor-pointer"
          >
            <button
              v-for="(role, i) in availableRoles"
              :key="role.key"
              type="button"
              class="relative px-4 py-3 inline-flex w-full rounded-lg focus:z-10 focus:outline-none focus:border-blue-300 focus:ring focus:ring-blue-200"
              :class="{
                'border-t border-gray-200 rounded-t-none': i > 0,
                'rounded-b-none': i !== Object.keys(availableRoles).length - 1,
              }"
              @click="updateRoleForm.role = role.key"
            >
              <div
                :class="{
                  'opacity-50':
                    updateRoleForm.role && updateRoleForm.role !== role.key,
                }"
              >
                <!-- Role Name -->
                <div class="flex items-center">
                  <div
                    class="text-sm text-gray-600 dark:text-gray-300"
                    :class="{
                      'font-semibold': updateRoleForm.role === role.key,
                    }"
                  >
                    {{ role.name }}
                  </div>

                  <svg
                    v-if="updateRoleForm.role == role.key"
                    class="ml-2 h-5 w-5 text-green-400"
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
                </div>

                <!-- Role Description -->
                <div class="mt-2 text-xs text-gray-600 dark:text-gray-300">
                  {{ role.description }}
                </div>
              </div>
            </button>
          </div>
        </div>
      </template>

      <template #footer>
        <SecondaryButton @click="currentlyManagingRole = false">
          Cancel
        </SecondaryButton>

        <PrimaryButton
          class="ml-3"
          :class="{ 'opacity-25': updateRoleForm.processing }"
          :disabled="updateRoleForm.processing"
          @click="updateRole"
        >
          Save
        </PrimaryButton>
      </template>
    </DialogModal>

    <!-- Leave Team Confirmation Modal -->
    <ConfirmationModal
      :show="confirmingLeavingTeam"
      @close="confirmingLeavingTeam = false"
    >
      <template #title> Leave Team </template>

      <template #content>
        Are you sure you would like to leave this team?
      </template>

      <template #footer>
        <SecondaryButton @click="confirmingLeavingTeam = false">
          Cancel
        </SecondaryButton>

        <DangerButton
          class="ml-3"
          :class="{ 'opacity-25': leaveTeamForm.processing }"
          :disabled="leaveTeamForm.processing"
          @click="leaveTeam"
          >Leave</DangerButton
        >
      </template>
    </ConfirmationModal>

    <!-- Remove Team Member Confirmation Modal -->
    <ConfirmationModal
      :show="!!teamMemberBeingRemoved"
      @close="teamMemberBeingRemoved = null"
    >
      <template #title> Remove Team Member </template>

      <template #content>
        Are you sure you would like to remove this person from the team?
      </template>

      <template #footer>
        <SecondaryButton @click="teamMemberBeingRemoved = null">
          Cancel
        </SecondaryButton>

        <DangerButton
          class="ml-3"
          :class="{ 'opacity-25': removeTeamMemberForm.processing }"
          :disabled="removeTeamMemberForm.processing"
          @click="removeTeamMember"
          >Remove</DangerButton
        >
      </template>
    </ConfirmationModal>
  </div>
</template>
