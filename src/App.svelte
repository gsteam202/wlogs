<script lang="ts">
	import isToday from 'date-fns/isToday';
	let server = '로크홀라';
	let characterName = '';

	let isLog = false;
	let logIndex = 0;
	let encounterName = '';
	let highPoint = '0';
	let logData = [];

	interface IData {
		characterID: number;
		characterName: string;
		encounterID: number;
		encounterName: string;
		percentile: number;
		startTime: number;
	}

	let highPointByBoss: Record<number, {
		encounterName: string;
		percentile: number;
	}> = {};

	const capitalizeFirstLetter = (str: string) => {
		return str.charAt(0).toUpperCase() + str.slice(1);
	}

	const getLogs = () => {
		fetch(`https://ko.classic.warcraftlogs.com/v1/parses/character/${characterName}/${server}/KR?api_key=99e3dac50bee26c6120cf6e073f02974`)
			.then((res) => res.json())
			.then((responseData) => {
				if (responseData?.status === 400) {
					window.alert('해당 캐릭터 정보를 찾을 수 없습니다. 다시 입력하세요.');
					return;
				}

				logData = responseData;
				calcLogs();
			});
	};

	const calcLogs = () => {
		logData.forEach((data: IData) => {
			if (isToday(new Date(data.startTime))) {
				highPointByBoss[data.encounterID] = {
					encounterName: data.encounterName,
					percentile: data.percentile,
				};
			}
		});

		isLog = true;
		if (Object.keys(highPointByBoss).length > 0) {
			loopLogs();
		} else {
			setTimeout(() => getLogs(), 5000);
		}
	};

	const loopLogs = () => {
		encounterName = highPointByBoss[logIndex].encounterName;
		highPoint = highPointByBoss[logIndex].percentile.toFixed(2);
		logIndex += 1;

		setTimeout(() => loopLogs(), 5000);
	};

	let handleSearch = async () => {
		if (!server || !characterName) {
			return window.alert('서버, 캐릭터명을 올바르게 입력/선택 하세요.');
		}

		getLogs();
	};
</script>
<main>
	<div class="wrap">
		{#if isLog}
			<p class="name">{server}/{capitalizeFirstLetter(characterName)}</p>
			{#if !encounterName}
				<p class="no-data">오늘 로그가 아직 없어요.</p>
			{:else}
				<p class="encounter-name">{encounterName}</p>
				<p class="high-point">{highPoint}</p>
			{/if}
		{:else}
			<p>
				<select bind:value={server}>
					<option>로크홀라</option>
					<option>라그나로스</option>
				</select>
			</p>
			<p>
				<input type="text" bind:value={characterName} placeholder="캐릭터명" />
			</p>
			<button on:click={handleSearch}>검색</button>
		{/if}
	</div>
</main>
<style>
</style>
