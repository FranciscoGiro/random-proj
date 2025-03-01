<script lang="ts">
	import {
		Chart as ChartJS,
		LineController,
		LineElement,
		LinearScale,
		PointElement,
		TimeScale,
		Title
	} from 'chart.js';
	import 'chartjs-adapter-date-fns';
	import annotationPlugin from 'chartjs-plugin-annotation';

	let chart: ChartJS | undefined;
	export let date: Date;
	export let minDate: Date;
	export let maxDate: Date;
	export let data: number[];
	console.log("date: " + date)

	const getDates = (): number[] => {
		let dateArray: number[] = [];
		var currentDate = minDate;
		while (currentDate <= maxDate) {
			dateArray.push(currentDate.valueOf());
			currentDate = new Date(currentDate.valueOf() + 86400000);
		}
		return dateArray;
	};

	const getAnnotations = (): any => {
		const alertDate = new Date('2020-07-03');
		const emergencyDate = new Date('2020-11-06');
		const calamityDate = new Date('2021-04-30');
		const finalDate = new Date('2021-05-30');
		return {
			currentDate: {
				type: 'line' as const,
				xMin: date.valueOf(),
				xMax: date.valueOf(),
				borderColor: 'rgb(0, 0, 0)',
				borderWidth: 2
			},
			alertLine: {
				type: 'line' as const,
				xMin: alertDate.valueOf(),
				xMax: alertDate.valueOf(),
				borderColor: 'rgba(0, 0, 0, 0.4)',
				borderWidth: 1
			},
			alertLabel:
				(alertDate > minDate && alertDate < maxDate) ||
				(emergencyDate > minDate && emergencyDate < maxDate) ||
				(alertDate < minDate && emergencyDate > maxDate)
					? {
							type: 'label' as const,
							content: 'Alerta',
							color: 'rgba(0, 0, 0, 0.4)',
							position: { x: 'center', y: 'start' },
							yMin: (c: any) => c.chart.scales.y.max,
							xMin: Math.max(alertDate.valueOf(), minDate.valueOf()),
							xMax: Math.min(emergencyDate.valueOf(), maxDate.valueOf())
					  }
					: undefined,
			emergencyLine: {
				type: 'line' as const,
				xMin: emergencyDate.valueOf(),
				xMax: emergencyDate.valueOf(),
				borderColor: 'rgba(0, 0, 0, 0.4)',
				borderWidth: 1
			},
			emergencyLabel:
				(emergencyDate > minDate && emergencyDate < maxDate) ||
				(calamityDate > minDate && calamityDate < maxDate) ||
				(emergencyDate < minDate && calamityDate > maxDate)
					? {
							type: 'label' as const,
							content: 'Emergência',
							color: 'rgba(0, 0, 0, 0.4)',
							position: { x: 'center', y: 'start' },
							yMin: (c: any) => c.chart.scales.y.max,
							xMin: Math.max(emergencyDate.valueOf(), minDate.valueOf()),
							xMax: Math.min(calamityDate.valueOf(), maxDate.valueOf())
					  }
					: undefined,
			calamityLine: {
				type: 'line' as const,
				xMin: calamityDate.valueOf(),
				xMax: calamityDate.valueOf(),
				borderColor: 'rgba(0, 0, 0, 0.4)',
				borderWidth: 1
			},
			calamityLabel:
				(calamityDate > minDate && calamityDate < maxDate) ||
				(finalDate > minDate && finalDate < maxDate) ||
				(calamityDate < minDate && finalDate > maxDate)
					? {
							type: 'label' as const,
							content: 'Calamidade',
							color: 'rgba(0, 0, 0, 0.4)',
							position: { x: 'center', y: 'start' },
							yMin: (c: any) => c.chart.scales.y.max,
							xMin: Math.max(calamityDate.valueOf(), minDate.valueOf()),
							xMax: Math.min(finalDate.valueOf(), maxDate.valueOf()),
							xAddjust: -50
					  }
					: undefined
		};
	};

	$: {
		if (chart && date) {
			chart!.options.plugins!.annotation!.annotations = getAnnotations();
			chart.data.datasets.at(0)!.data = data;
		}
	}

	$: chart?.update();

	const config = {
		type: 'line' as const,
		data: {
			labels: getDates(),
			datasets: [
				{
					borderColor: 'rgb(37, 99, 235)',
					pointRadius: 0,
					pointHoverRadius: 0,
					data: data
				}
			]
		},
		options: {
			responsive: true,
			scales: {
				x: {
					font: {
						size: 4
					},
					type: 'time' as const,
					time: {
						displayFormats: {
							day: 'dd MMM',
							month: 'dd MMM'
						}
					},
					min: minDate.valueOf(),
					max: maxDate.valueOf(),
					title: {
						display: true,
						text: 'Tempo',
						padding: 0
					}
				},
				y: {
					font: {
						size: 4
					},
					title: {
						display: true,
						text: 'Tx. inc. cum. (* 10\u2075 hab.)    ',
						padding: 0
					}
				}
			},
			plugins: {
				title: {
					display: true,
					text: 'Incidência cumulativa a nível nacional',
					padding: {
						top: 0,
						bottom: 15
					}
				},
				legend: {
					display: false
				},
				tooltip: {
					enabled: false
				},
				annotation: {
					annotations: getAnnotations()
				}
			}
		}
	};

	function initialize(node: HTMLCanvasElement) {
		ChartJS.register(
			Title,
			LineController,
			LineElement,
			PointElement,
			LinearScale,
			TimeScale,
			annotationPlugin
		);
		const ctx = node.getContext('2d') as CanvasRenderingContext2D;
		chart = new ChartJS(ctx, config);
		return {
			destroy() {
				chart?.destroy();
			}
		};
	}
</script>

<div class="bg-white border border-gray-200 rounded-lg shadow-sm p-4">
	<canvas use:initialize />
</div>
