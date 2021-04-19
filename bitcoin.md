<!doctype html>
<html lang="en">
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style>
			@font-face {
				font-family: "Trebuchet MS";
				font-style: normal;
				font-weight: normal;
				src: local("Trebuchet MS"), url("trebuc.woff") format("woff");
			}

			.ticker {
				width: 100%;
				padding: 50px;
				text-align: center;
				box-sizing: border-box;
				font-family: "Trebuchet MS"
			}

			.symbol {
				color: #5d5d5d;
				font-size: 9em;
			}

			.stat {
				color: #787878;
				top:10px;
				position:relative;
			}

			.stat_value, .stat {
				font-size: 6em;
			}

			.stat_value::before, .price::before {
				content: "$";
			}

			.price {
				font-size: 12em;
			}
			
			.footer {
				position: absolute; 
				bottom: 0;
				right: 0;
				font-size: 0.6em;
				padding-right:2px;
			}
		</style>
	</head>
	<body>
		<div class="ticker">
			<div class="symbol">BTC</div>
			<div class="price" id="price"></div>
			<div class="stat">change</div> <div class="stat_value" id="change"></div>
			<div class="stat">low</div> <div class="stat_value" id="low"></div>
			<div class="stat">high</div> <div class="stat_value" id="high"></div>
		</div>
		<div class="footer">
			<span id="battery"></span>
			<span id="temp"></span>
		</div>
		<script>
		function ajax(url, callback) {
			var req = new XMLHttpRequest;
			req.onreadystatechange = function() {
				if (req.readyState == 4 && req.status == 200) {
					callback(JSON.parse(req.responseText));
				}
			}
			req.open("POST", url, true);
			req.send();
		}

		function updateById(element, text) {
			document.getElementById(element).textContent = text
		}

		function numberWithCommas(x) {
			return x.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
		}

		function getSearchParameters() {
			var prmstr = window.location.search.substr(1);
			return prmstr != null && prmstr != "" ? transformToAssocArray(prmstr) : {};
		}

		function transformToAssocArray( prmstr ) {
			var params = {};
			var prmarr = prmstr.split("&");
			for ( var i = 0; i < prmarr.length; i++) {
				var tmparr = prmarr[i].split("=");
				params[tmparr[0]] = tmparr[1];
			}
			return params;
		}
		
		var getConfig = getSearchParameters(); 

		function ticker() {
			ajax("https://www.bitstamp.net/api/ticker/btcusd", function (data) {
				updateById("price", numberWithCommas(data.bid));
				updateById("change", (data.bid > data.open ? "+" : "") + numberWithCommas((data.bid-data.open).toFixed(2)) + " (" + (((data.bid-data.open)/data.bid) * 100).toFixed(2) + "%)");
				updateById("low", numberWithCommas(data.low));
				updateById("high", numberWithCommas(data.high));
			})
		};

		var updateFreq = 90; // Refresh ticker values every n seconds
		if(getConfig.hasOwnProperty('updateFreq')) {
			updateFreq = getConfig.updateFreq;
		}

		if(updateFreq > 0) {
			setInterval("ticker()", updateFreq*1000);
		}
		ticker();

		function deviceInfo() {
			var s = okular.DevicesStatus()[okular.session_uuid]
			updateById("battery", okular.BatteryLevel + "%")
			updateById("temp", okular.Temperature + "°C")
		}

		var deviceFreq = 300; // Refresh battery and temperature every n seconds
		if(getConfig.hasOwnProperty('deviceFreq')) {
			deviceFreq = getConfig.deviceFreq;
		}

		if(typeof(okular) !== "undefined") {
			if(deviceFreq > 0) {
				setInterval("deviceInfo()", deviceFreq*1000);
			}
			deviceInfo();
		}
		</script>
	</body>
</html>
