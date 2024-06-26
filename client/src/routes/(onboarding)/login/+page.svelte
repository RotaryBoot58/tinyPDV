<script lang="ts">
	import * as Form from "$lib/components/ui/form";
	import { Input } from "$lib/components/ui/input";
	import {
		type SuperValidated,
		type Infer,
		superForm,
	} from "sveltekit-superforms";

	import { zodClient } from "sveltekit-superforms/adapters";
	import {
		newLoginSchema,
		type NewLoginSchema,
	} from "$lib/schemas/newLoginSchema";
	import Button from "$lib/components/ui/button/button.svelte";
	import { Rocket } from "lucide-svelte";
	import { apiService } from "$lib/services/ApiService";

	export let data: SuperValidated<Infer<NewLoginSchema>>;

	const form = superForm(data, {
		SPA: true,
		validators: zodClient(newLoginSchema),
		onUpdated({ form }) {
			if (form.valid) {
				doLogin(form.data);
			}
		},
	});

	const { form: formData, enhance } = form;

	$formData.email = "test@example.com";
	$formData.password = "password";

	const doLogin = async (values: object) => {
		try {
			await apiService.post("login", values);
			let { data } = await apiService.get("api/user");

			if (data.id) {
				localStorage.setItem("user", JSON.stringify(data));
				window.location.href = "/";
			}
		} catch (error: any) {
			apiError = error.response.data.message;
		}
	};

	let apiError: any = null;
</script>

<div
	class="h-screen w-screen flex flex-col justify-center items-center bg-no-repeat bg-cover bg-center bg-opacity-5"
>
	<h1 class=" flex text-3xl font-extrabold mb-2">
		<Rocket class="mt-2 mr-2" /> tinyPDV
	</h1>
	<div
		class="w-80 md:w-96 rounded-xl border border-slate-600 border-opacity-30 bg-card bg-opacity-100 text-card-foreground"
	>
		<div class="flex flex-col p-6 space-y-1">
			<h3 class="font-semibold tracking-tight text-2xl">Accesse a sua conta</h3>
			<p class="text-sm text-muted-foreground">Informe o seu email e senha</p>
		</div>

		<form method="POST" use:enhance class="p-6 pt-0 grid gap-4">
			<Form.Field {form} name="email" class="grid gap-2">
				<Form.Control let:attrs>
					<Form.Label
						class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
						>Email</Form.Label
					>
					<Input
						{...attrs}
						bind:value={$formData.email}
						placeholder="Digite seu e-mail"
						autocomplete="none"
					/>
				</Form.Control>
				<Form.FieldErrors class="text-red-500" />
			</Form.Field>

			<Form.Field {form} name="password" class="grid gap-2">
				<Form.Control let:attrs>
					<Form.Label
						class="text-sm font-medium leading-none peer-disabled:cursor-not-allowed peer-disabled:opacity-70"
						>Senha</Form.Label
					>
					<Input
						{...attrs}
						bind:value={$formData.password}
						class="flex h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm shadow-sm transition-colors file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:cursor-not-allowed disabled:opacity-50"
						placeholder="******"
						type="password"
					/>
				</Form.Control>
				<Form.FieldErrors class="text-red-500" />
			</Form.Field>

			{#if apiError}
				<p class="text-red-500 text-sm">{apiError}</p>
			{/if}

			<Button type="submit">Iniciar Sessão</Button>
		</form>

		<fieldset class="px-6 py-4">
			<legend class="text-sm text-muted-foreground">
				Ainda não tem uma conta? <a
					href="/register"
					class="text-accent-foreground">Registre-se</a
				>
			</legend>
		</fieldset>
	</div>
</div>
