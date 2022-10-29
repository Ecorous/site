<script lang="ts">
	import { browser } from '$app/environment';
	import PKAPI from 'pkapi.js';
	import { onMount } from 'svelte';
	const api = new PKAPI({
		base_url: 'https://api.pluralkit.me',
		version: 2,
		token: undefined
	});
	// @ts-ignore I don't why but vite is screaming at every type annotation
	let system;
	// @ts-ignore again types
	let members;
	let fronters;
	onMount(async () => {
		system = await api.getSystem({ system: 'rraib' });
		members = await api.getMembers({ system: 'rraib' });
		fronters = await api.getFronters({ system: 'rraib' });
	});
	let hostname = '';
	let pathname = '/';
	if (browser) {
		if (window.location.pathname == '/') {
			pathname = 'root';
		} else pathname = window.location.pathname;
		hostname = pathname + '@' + window.location.hostname + ' >';
	}

	// @ts-ignore
	function respondCommand(command, toAdd, clearFirst = false) {
		let out = document.getElementById('prevText');
		if (out == null) {
			console.error('An error has occurred. out == null');
			return;
		}
		if (clearFirst) {
			out.innerHTML = '';
		}
		out.innerHTML += '<p> ' + hostname + ' ' + command + '</p> ' + toAdd;
	}
	// @ts-ignore - either we get an error here, or the site doesn't load, so I picked the error here :P
	function test(event) {
		if (!browser) {
			console.log('hi');
			return;
		}
		if (event.key == 'Enter') {
			let input = document.getElementById('inputID');
			// @ts-ignore - I don't why this errors, but in the end it works
			let command = input.value;
			let subcommand = command.split(' ')[1];
			command = command.split(' ')[0];
			if (command == 'about') {
				//@ts-ignore - again, vite is erroring at every type annotation
				respondCommand(
					command,
					//@ts-ignore
					'Hello! We are the Ecorous System! There are ' + members.size + ' members within us</p>'
				);
			} else if (command == 'clear') {
				respondCommand(command, '<i> Console cleared </i>', true);
			} else if (command == 'help') {
				respondCommand(
					command,
					`<p>List of commands</p>
										 <ul>
											<li>about</li>
											<li>clear</li>
											<li>help</li>
											<li>page <ul>
													 <li>help</li>
													 <li>reload: refresh</li>
													 <li>back: <-</li>
													 <li>forward: -></li>	   
												     </ul>
											</li>
											<li>system <ul>
													   <li>help</li>
													   <li>members: m</li>
													   <li>fronter: f</li>
											</li>
										</ul>`
				);
			} else if (command == 'page') {
				if (subcommand == 'help') {
					respondCommand(
						// @ts-ignore
						input.value,
						`page <ul>
									<li>help</li>
									<li>reload: refresh</li>
									<li>back: <-</li>
									<li>forward: -></li>	   
							 </ul>
						`
					);
				} else if (subcommand == 'reload' || subcommand == 'refresh') {
					//@ts-ignore
					respondCommand(input.value, `Reloading...`);
					window.location.reload();
				} else if (subcommand == 'back' || subcommand == '<-') {
					//@ts-ignore
					respondCommand(input.value, 'Going back...');
					window.history.back();
				} else if (subcommand == 'forward' || subcommand == '->') {
					//@ts-ignore
					respondCommand(input.value, 'Going forwards...');
					window.history.forward();
				} else {
					respondCommand(
						// @ts-ignore
						input.value,
						`page <ul>
									<li>help</li>
									<li>reload: refresh</li>
									<li>back: <-</li>
									<li>forward: -></li>	   
							 </ul>
						`
					);
				}
			} else if (command == 'system') {
				if (subcommand == 'members' || subcommand == 'm') {
					let memberString = '<p> List of members </p> <ul>';
					//@ts-ignore type annotations
					members.forEach((member) => {
						memberString += '<li>' + member.name + '</li>';
					});

					memberString += '</ul>';
					//@ts-ignore
					respondCommand(input.value, memberString);
				} else if (subcommand == 'fronter' || subcommand == 'fronters' || subcommand == 'f') {
					let frontersString = '<p> Fronter(s) </p> <ul>';
					//@ts-ignore
					fronters.members.forEach((m) => {
						frontersString += '<li>' + m.name + '</li>';
					});
					frontersString += '</ul>';
					//@ts-ignore
					respondCommand(input.value, frontersString);
				} else {
					respondCommand(
						input.value,
						`<li>system <ul>
													   <li>help</li>
													   <li>members: m</li>
													   <li>fronter: f</li>
												</li>`
					);
				}
			} else {
				respondCommand(command, "Unknown Command. Type 'help' for a list of commands");
			}
			// @ts-ignore
			input.value = '';
		}
	}
</script>

<div id="prevText">
	Hello! Please type `help` for a list of commands. And no, this is not a real shell, it is merely javascript code. Don't try anything stupid
</div>
<!-- svelte-ignore a11y-missing-attribute -->
<a>{hostname} </a><input id="inputID" on:keydown={test} class="inClass" />

<style>
	.inClass {
		background-color: rgb(6, 6, 6);
		border: none;
		border-radius: 15px;
	}

	* {
		color: green;
		font-family: monospace;
	}
	textarea:focus,
	input:focus {
		outline: none;
	}
	:global(body) {
		background-color: black;
	}
</style>