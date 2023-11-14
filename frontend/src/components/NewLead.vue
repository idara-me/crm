<template>
  <div class="flex flex-col gap-4">
    <div v-for="section in allFields" :key="section.section">
      <div class="grid grid-cols-3 gap-4">
        <div 
          v-for="field in section.fields" 
          :key="field.name" 
          :class="{'col-span-3': field.name === 'notes', 'col-span-1': field.name !== 'notes'}"
        >
          <div v-if="field.name === 'notes'">
            <div class="mb-2 text-sm text-gray-600">{{ field.label }}</div>
            <textarea v-model="newLead[field.name]" class="form-textarea w-full"></textarea>
          </div>
          <div v-else>
            <div class="mb-2 text-sm text-gray-600">{{ field.label }}</div>
            <FormControl
              v-if="field.type === 'select'"
              type="select"
              :options="field.options"
              v-model="newLead[field.name]"
            >
              <template v-if="field.name == 'status'" #prefix>
                <IndicatorIcon :class="leadStatuses[newLead[field.name]].color" />
              </template>
            </FormControl>
            <FormControl
              v-else-if="field.type === 'email'"
              type="email"
              v-model="newLead[field.name]"
            />
            <FormControl
              v-else-if="field.type === 'link'"
              type="autocomplete"
              :value="newLead[field.name]"
              :options="field.options"
              @change="(e) => field.change(e)"
              :placeholder="field.placeholder"
              class="form-control"
            />
            <FormControl
              v-else-if="field.type === 'user'"
              type="autocomplete"
              :options="activeAgents"
              :value="getUser(newLead[field.name]).full_name"
              @change="(option) => (newLead[field.name] = option.email)"
              :placeholder="field.placeholder"
            >
              <template #prefix>
                <UserAvatar class="mr-2" :user="newLead[field.name]" size="sm" />
              </template>
              <template #item-prefix="{ option }">
                <UserAvatar class="mr-2" :user="option.email" size="sm" />
              </template>
            </FormControl>
            <Dropdown
              v-else-if="field.type === 'dropdown'"
              :options="statusDropdownOptions(newLead)"
              class="w-full flex-1"
            >
              <template #default="{ open }">
                <Button
                  :label="newLead[field.name]"
                  class="w-full justify-between"
                >
                  <template #prefix>
                    <IndicatorIcon
                      :class="leadStatuses[newLead[field.name]].color"
                    />
                  </template>
                  <template #default>{{ newLead[field.name] }}</template>
                  <template #suffix>
                    <FeatherIcon
                      :name="open ? 'chevron-up' : 'chevron-down'"
                      class="h-4 text-gray-600"
                    />
                  </template>
                </Button>
              </template>
            </Dropdown>
            <FormControl v-else type="text" v-model="newLead[field.name]" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import IndicatorIcon from '@/components/Icons/IndicatorIcon.vue'
import UserAvatar from '@/components/UserAvatar.vue'
import { usersStore } from '@/stores/users'
import { organizationsStore } from '@/stores/organizations'
import { leadStatuses, statusDropdownOptions, activeAgents } from '@/utils'
import { FormControl, Button, Dropdown, FeatherIcon } from 'frappe-ui'

const { getUser } = usersStore()
const { getOrganizationOptions } = organizationsStore()

const props = defineProps({
  newLead: {
    type: Object,
    required: true,
  },
})

const allFields = [
  {
    section: 'Lead Details',
    fields: [
      {
        label: 'First Name',
        name: 'first_name',
        type: 'data',
      },
      {
        label: 'Last Name',
        name: 'last_name',
        type: 'data',
      },
      {
        label: 'City',
        name: 'city',
        type: 'data',
      },
    ],
  },
  {
    section: 'Contact Details',
    fields: [
      {
        label: 'Email',
        name: 'email',
        type: 'data',
      },
      {
        label: 'Phone Number',
        name: 'phone',
        type: 'data',
      },
    ],
  },
  {
    section: 'Details',
    fields: [
      {
        label: 'How did you hear about us',
        name: 'how_did_you_hear_about_us',
        type: 'select',
        options: [
          {
            label: 'Facebook',
            value: 'Facebook',
          },
          {
            label: 'Instagram',
            value: 'Instagram',
          },
          {
            label: 'Word of Mouth',
            value: 'Word of Mouth',
          },
        ],
      },
       {
        label: 'Interested Vehicle',
        name: 'interested_vehicle',
        type: 'data',
      },
      {
        label: 'Payment',
        name: 'payment',
        type: 'select',
        options: [
          {
            label: 'Cash',
            value: 'Cash',
          },
          {
            label: 'Cheque',
            value: 'Cheque',
          },
          {
            label: 'Cash & Cheque',
            value: 'Cash/Cheque',
          },
        ],
      },
    ],
  },
  {
    section: 'Details',
    fields: [
      {
        label: 'Number of visits',
        name: 'number_of_visits',
        type: 'int',
      },
      {
        label: 'Budget (optional)',
        name: 'budget',
        type: 'currency',
        required: 0,
      },
      {
        label: 'Lead Score (Serious)',
        name: 'lead_score',
        type: 'select',
        options: [
          { label: '1', value: '1' },
          { label: '2', value: '2' },
          { label: '3', value: '3' },
          { label: '4', value: '4' },
          { label: '5', value: '5' },
        ],
      },
    ],
  },
  {
  section: 'Other Details',
  fields: [
    {
      label: 'Status',
      name: 'status',
      type: 'select',
      options: statusDropdownOptions(props.newLead),
    },
    {
      label: 'Lead owner',
      name: 'lead_owner',
      type: 'link',
      options: 'User',
    },
    {
      label: 'Assigned To',
      name: 'assigned_to',
      type: 'link',
      options: 'User',
    },
  ],
},
  {
    section: 'Notes',
    fields: [
      {
        label: 'Notes',
        name: 'notes',
        type: 'textarea',
      },
    ],
  },
]
</script>