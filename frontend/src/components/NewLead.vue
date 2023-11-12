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
            <Autocomplete
              v-else-if="field.type === 'link'"
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
            </Autocomplete>
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
            <FormControl 
              v-else 
              :type="field.type === 'data' ? 'text' : field.type" 
              v-model="newLead[field.name]" 
            />
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
import { leadStatuses, statusDropdownOptions, activeAgents } from '@/utils'
import {
  FormControl,
  Button,
  Autocomplete,
  Dropdown,
  FeatherIcon,
} from 'frappe-ui'
import { computed } from 'vue'

const { getUser, users } = usersStore()
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
        label: 'Country Code',
        name: 'country_code',
        type: 'select',
        options: [
          { label: '+970', value: '+970' },
          { label: '+972', value: '+972' },
        ],
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
      {
        label: 'Number of visits',
        name: 'number_of_visits',
        type: 'int',
      },
      {
        label: 'Budget (optional)',
        name: 'budget',
        type: 'currency',
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
        options: [
          { label: 'Open', value: 'Open' },
          { label: 'Contacted', value: 'Contacted' },
          { label: 'Nurture', value: 'Nurture' },
          { label: 'Qualified', value: 'Qualified' },
          { label: 'Unqualified', value: 'Unqualified' },
          { label: 'Junk', value: 'Junk' },
        ],
      },
      {
        label: 'Lead owner',
        name: 'lead_owner',
        type: 'link',
        placeholder: 'User',
      },
      {
        label: 'Assigned To',
        name: 'assigned_to',
        type: 'link',
        placeholder: 'User',
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
