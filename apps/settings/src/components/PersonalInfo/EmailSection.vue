<!--
  - @copyright 2021 Christopher Ng <chrng8@gmail.com>
  -
  - @author 2021 Christopher Ng <chrng8@gmail.com>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program.  If not, see <http://www.gnu.org/licenses/>.
  -->

<template>
	<form id="emailform" class="section">
		<h3>
			<label for="email">{{ t('settings', 'Email') }}</label>
			<a href="#" class="federation-menu" :aria-label="t('settings', 'Change privacy level of email')">
				<span class="icon-federation-menu icon-password">
					<span class="icon-triangle-s"></span>
				</span>
			</a>
			<Actions class="actions">
				<ActionButton icon="icon-add" @click.stop.prevent="addAdditionalEmail">
					{{ t('settings', 'Add email address') }}
				</ActionButton>
			</Actions>
		</h3>
		<div
			class="verify"
			:class="{ hidden: hideVerification }">
			<img
				id="verify-email"
				:src="verificationIcon"
				:title="email.message"
				:data-status="email.verified" />
		</div>
		<input
			type="email"
			name="email"
			id="email"
			v-model="email.value"
			@keyup.enter.stop.prevent="updateEmails"
			:class="{ hidden: hideEmailInput }"
			:placeholder="t('settings', 'Your primary email address')"
			autocomplete="on"
			autocapitalize="none"
			autocorrect="off" />
		<span class="icon-checkmark hidden"></span>
		<span class="icon-error hidden"></span>
		<span v-if="hideEmailInput">{{ email.value ? email.value : t('settings', 'No email address set') }}</span>
		<em v-if="!hideEmailInput">{{ t('settings', 'For password reset and notifications') }}</em>
		<input type="hidden" id="emailscope" v-model="email.scope" />
		<input v-for="(email, index) in additionalEmails"
			type="email"
			name="additionalEmail[]"
			:id="`additionalEmail-${index}`"
			v-model="email.value"
			@keyup.enter.stop.prevent="updateEmails"
			:class="{ hidden: hideEmailInput }"
			:placeholder="t('settings', `Additional email address ${index+1}`)"
			:key="index"
			autocomplete="on"
			autocapitalize="none"
			autocorrect="off" />
	</form>
</template>

<script>
import { Actions, ActionButton } from '@nextcloud/vue'
import { imagePath } from '@nextcloud/router'
// import { showError } from '@nextcloud/dialogs'
import { setPrimaryEmail, setAdditionalEmails } from '../../service/PersonalInfoService'

export default {
	name: 'EmailSection',
	components: {
		Actions,
		ActionButton,
	},
	props: {
		initialEmails: {
			type: Object,
			required: true,
		},
		accountParams: {
			type: Object,
			required: true,
		},
		VerificationEnum: {
			type: Object,
			required: true,
		},
	},
	computed: {
		verificationIcon() {
			const verificationIconMap = {
				[this.VerificationEnum.VERIFIED]: imagePath('core', 'actions/verified.svg'),
				[this.VerificationEnum.VERIFICATION_IN_PROGRESS]: imagePath('core', 'actions/verifying.svg'),
				[this.VerificationEnum.NOT_VERIFIED]: imagePath('core', 'actions/verify.svg'),
				default: imagePath('core', 'actions/verify.svg'),
			}

			return Object.prototype.hasOwnProperty.call(verificationIconMap, this.email.verified) ? verificationIconMap[this.email.verified] : verificationIconMap.default
		}
	},
	data() {
		/* eslint-disable */
		console.log(this.initialEmails)
		console.log(this.accountParams)
		return {
			email: this.initialEmails.primaryEmail,
			hideVerification: this.initialEmails.primaryEmail.value === '' || this.initialEmails.primaryEmail.scope !== 'public',
			hideEmailInput: !this.accountParams.displayNameChangeSupported,
			additionalEmails: this.initialEmails.additionalEmails,
		}
	},
	methods: {
		async addAdditionalEmail() {
			if (this.additionalEmails.every(({ value }) => value !== '')) this.additionalEmails.push({ value: '' })
		},

		async updateEmails() {
			try {
				const data = await setPrimaryEmail(this.email.value)
				console.log(data.ocs.meta)
			} catch (e) {
				console.error('Unable to update primary email address', e)
				// TODO fix error dialog styles
				// showError(t('settings', 'An error happened while updating the primary email address'))
			}

			try {
				const data = await setAdditionalEmails(this.additionalEmails.map(({ value }) => value))
				console.log(data.ocs.meta)
			} catch (e) {
				console.error('Unable to update additional email addresses', e)
				// showError(t('settings', 'An error happened while updating additional email addresses'))
			}
		},
	}
}
</script>

<style scoped lang="scss">
	.actions {
		// TODO do this the right way
		margin: -12px 0 0 2px !important;
	}
</style>
