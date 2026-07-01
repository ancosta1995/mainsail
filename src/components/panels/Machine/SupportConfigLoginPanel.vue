<template>
    <v-row align="center" justify="center" class="fill-height py-8">
        <v-col cols="12" sm="8" md="5" lg="4">
            <v-alert type="warning" prominent border="left" class="mb-4">
                Área restrita ao suporte técnico. Alterações indevidas podem danificar a impressora.
            </v-alert>
            <v-card>
                <v-card-title>Acesso à configuração</v-card-title>
                <v-card-text>
                    <v-form @submit.prevent="login">
                        <v-text-field
                            v-model="username"
                            label="Usuário"
                            autocomplete="username"
                            :disabled="loading"
                            required />
                        <v-text-field
                            v-model="password"
                            label="Senha"
                            type="password"
                            autocomplete="current-password"
                            :disabled="loading"
                            required />
                        <v-alert v-if="errorMessage" type="error" dense class="mt-2">
                            {{ errorMessage }}
                        </v-alert>
                    </v-form>
                </v-card-text>
                <v-card-actions>
                    <v-spacer />
                    <v-btn color="primary" :loading="loading" @click="login">Entrar</v-btn>
                </v-card-actions>
            </v-card>
        </v-col>
    </v-row>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import type { ConfigJson } from '@/store/types'

const DEFAULT_SUPPORT_USER = 'biqu'
const DEFAULT_SUPPORT_PASSWORD = 'biqu'

@Component
export default class SupportConfigLoginPanel extends Mixins(BaseMixin) {
    username = ''
    password = ''
    loading = false
    errorMessage = ''

    async login() {
        this.errorMessage = ''
        this.loading = true

        try {
            const expected = await this.getExpectedCredentials()

            if (this.username === expected.user && this.password === expected.password) {
                this.$emit('authenticated')
                return
            }

            this.errorMessage = 'Usuário ou senha incorretos.'
        } finally {
            this.loading = false
        }
    }

    async getExpectedCredentials(): Promise<{ user: string; password: string }> {
        const base = import.meta.env.BASE_URL ?? '/'

        try {
            const response = await fetch(`${base}config.json`)
            if (!response.ok) {
                return { user: DEFAULT_SUPPORT_USER, password: DEFAULT_SUPPORT_PASSWORD }
            }

            const config = (await response.json()) as ConfigJson
            return {
                user: config.supportConfigUser ?? DEFAULT_SUPPORT_USER,
                password: config.supportConfigPassword ?? DEFAULT_SUPPORT_PASSWORD,
            }
        } catch {
            return { user: DEFAULT_SUPPORT_USER, password: DEFAULT_SUPPORT_PASSWORD }
        }
    }
}
</script>
