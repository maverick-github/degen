<script>
	import axios from 'axios';
	import moment from 'moment';
	import '@fortawesome/fontawesome-free/js/all.js';
	import 'bootstrap/dist/css/bootstrap.css';

	import { onMount } from 'svelte';

	/**
	 * @type {any[]}
	 */
	let tokens = [];

	onMount(async () => {
		const response = await axios.post(
			'https://graph.defined.fi/graphql',
			{
				query: `{
			filterTokens(
			  filters: {      
				liquidity:{gt:1000}
				change24:{gt:1}
				uniqueTransactions24:{gt:50}
				network: [1]
			  }
			  rankings:{
				attribute:createdAt
				direction:DESC
			  }
			  limit: 14
			) {
			  results {
				token {
				  name
				  symbol
				  address
				}
				priceUSD
				createdAt
				change24
				uniqueTransactions24
				volume24
				liquidity
				marketCap
			  }
			}
		  }`
			},
			{
				headers: {
					'Content-Type': 'application/json',
					Authorization: '34b3af504a760b3a97b471e842393280b80fdb8d'
				}
			}
		);

		tokens = response.data.data.filterTokens.results;
		tokens.forEach((token) => {
			let now = Date.now(); // Current time in Unix timestamp
			let age = moment.duration(now - token.createdAt * 1000); // Subtract token.createdAt from current time
			let y = age.years();
			let mo = age.months();
			let d = age.days();
			let h = age.hours();
			let m = age.minutes();
			token.age =
				(y > 0 ? y + 'y ' : '') +
				(mo > 0 ? mo + 'mo ' : '') +
				(d > 0 ? d + 'd ' : '') +
				(h > 0 ? h + 'h ' : '') +
				(m > 0 ? m + 'm ' : '');
		});
	});
</script>

<table class="table table-striped table-bordered table-hover table-responsive">
	<thead>
		<tr>
			<th scope="col">Name/Symbol</th>
			<th scope="col">Moonarch</th>
			<th scope="col">Honeypot.is</th>
			<th scope="col">BubbleMap</th>
			<th scope="col">Age</th>
			<th scope="col">Price</th>
			<th scope="col">24h</th>
			<th scope="col">Uniq Tx</th>
			<th scope="col">Volume</th>
			<th scope="col">Liquidity</th>
			<th scope="col">MarketCap</th>
		</tr>
	</thead>
	<tbody>
		{#each tokens as token (token.token.name)}
			<tr>
				<td>
					<button	on:click={() => navigator.clipboard.writeText(token.token.address)}><i class="fas fa-copy" /></button>
					{token.token.symbol}<br><span class="text-sm-start">{token.token.name}</span>
				</td>
				<td><a href={`https://eth.moonarch.app/token/${token.token.address}`} target="_blank"><i class="fas fa-external-link-alt" /></td>
				<td><a href={`https://honeypot.is/ethereum?address=${token.token.address}`} target="_blank"><i class="fas fa-external-link-alt" /></td>
				<td><a href={`https://tokensniffer.com/bubble/eth/${token.token.address}`} target="_blank"><i class="fas fa-external-link-alt" /></td>
				<td>{token.age}</td>
				<td>{token.priceUSD}</td>
				<td>{Math.floor(token.change24 * 100)}%</td>
				<td>{token.uniqueTransactions24}</td>
				<td>{token.volume24}</td>
				<td>{token.liquidity}</td>
				<td>{token.marketCap}</td>
			</tr>
		{/each}
	</tbody>
</table>
