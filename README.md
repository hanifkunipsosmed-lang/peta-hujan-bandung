<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Peta Sebaran Stasiun Hujan Kota Bandung</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <style>
        #map {
            height: 500px;
            width: 100%;
            border: 1px solid #ccc;
            border-radius: 8px;
        }
        
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        
        h1 {
            color: #2c3e50;
            text-align: center;
        }
        
        .info {
            background-color: white;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        /* Modifikasi-3: Legenda */
        .legend {
            background: white;
            padding: 10px 14px;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
            font-size: 13px;
            line-height: 1.8;
            min-width: 200px;
        }

        .legend h4 {
            margin: 0 0 8px 0;
            font-size: 13px;
            color: #2c3e50;
            border-bottom: 1px solid #ddd;
            padding-bottom: 4px;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 4px;
        }

        .legend-dot {
            width: 14px;
            height: 14px;
            border-radius: 50%;
            display: inline-block;
            border: 1px solid rgba(0,0,0,0.3);
            flex-shrink: 0;
        }

        /* Modifikasi-5: Search Box */
        .search-container {
            position: absolute;
            top: 10px;
            left: 60px;
            z-index: 1000;
            display: flex;
            gap: 4px;
        }

        #searchInput {
            padding: 7px 12px;
            border: 2px solid #3498db;
            border-radius: 6px;
            font-size: 13px;
            width: 220px;
            outline: none;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
        }

        #searchInput:focus {
            border-color: #2980b9;
            box-shadow: 0 2px 10px rgba(52,152,219,0.4);
        }

        #searchBtn {
            padding: 7px 12px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 13px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
        }

        #searchBtn:hover {
            background: #2980b9;
        }
    </style>
</head>
<body>
    <!-- Modifikasi-1: Heading diubah -->
    <h1>🌧️ Peta Sebaran Stasiun Hujan & Curah Hujan Bandung</h1>
    <div class="info">
        <!-- Modifikasi-1: Teks info panel diubah -->
        <strong>ℹ️ Informasi:</strong> Klik pada marker stasiun hujan untuk melihat data curah hujan
    </div>
    <div id="map"></div>

    <script>
        // =============================================
        // Modifikasi-1: Koordinat awal = pusat Kota Bandung, zoom 13
        // =============================================
        var map = L.map('map').setView([-6.9175, 107.6191], 13);

        // =============================================
        // Modifikasi-4: Layer Control (Basemap)
        // =============================================
        var osmLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            maxZoom: 19
        });

        var googleSatellite = L.tileLayer('https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
            attribution: '&copy; Google Maps',
            maxZoom: 20
        });

        var googleTerrain = L.tileLayer('https://mt1.google.com/vt/lyrs=p&x={x}&y={y}&z={z}', {
            attribution: '&copy; Google Maps',
            maxZoom: 20
        });

        // Tambahkan OSM sebagai default
        osmLayer.addTo(map);

        var baseMaps = {
            "OpenStreetMap": osmLayer,
            "Google Maps Satellite": googleSatellite,
            "Google Maps Terrain": googleTerrain
        };

        L.control.layers(baseMaps).addTo(map);

        // =============================================
        // Tambahkan scale bar
        // =============================================
        L.control.scale({
            metric: true,
            imperial: false
        }).addTo(map);

        // Pindahkan kontrol zoom ke kiri bawah
        map.zoomControl.setPosition('bottomleft');

        // =============================================
        // Modifikasi-2: Data 5 Stasiun Hujan Kota Bandung
        // =============================================
        var stasiunData = [
            {
                nama: "Stasiun Hujan Gedebage",
                koordinat: [-6.9439, 107.7082],
                ketinggian: 675,
                curahHujan: 2150
            },
            {
                nama: "Stasiun Hujan Dayeuhkolot",
                koordinat: [-6.9794, 107.6183],
                ketinggian: 660,
                curahHujan: 2080
            },
            {
                nama: "Stasiun Hujan Lembang",
                koordinat: [-6.8126, 107.6156],
                ketinggian: 1250,
                curahHujan: 2800
            },
            {
                nama: "Stasiun Hujan Cimahi",
                koordinat: [-6.8722, 107.5423],
                ketinggian: 740,
                curahHujan: 2200
            },
            {
                nama: "Stasiun Hujan Ujungberung",
                koordinat: [-6.9050, 107.6850],
                ketinggian: 685,
                curahHujan: 2100
            }
        ];

        // Fungsi warna marker berdasarkan curah hujan
        function getMarkerColor(curahHujan) {
            if (curahHujan > 2500) {
                return 'red';
            } else if (curahHujan >= 2100) {
                return 'orange';
            } else {
                return 'green';
            }
        }

        // Fungsi membuat circle marker
        function createMarker(stasiun) {
            var color = getMarkerColor(stasiun.curahHujan);

            var marker = L.circleMarker(stasiun.koordinat, {
                radius: 10,
                fillColor: color,
                color: '#333',
                weight: 1.5,
                opacity: 1,
                fillOpacity: 0.85
            }).addTo(map);

            // Popup berisi Nama Stasiun, Ketinggian, Curah Hujan Tahunan
            marker.bindPopup(
                '<b>📍 ' + stasiun.nama + '</b><br>' +
                '⛰️ Ketinggian: ' + stasiun.ketinggian + ' mdpl<br>' +
                '🌧️ Curah Hujan Tahunan: ' + stasiun.curahHujan + ' mm/thn'
            );

            // Tooltip muncul saat hover
            marker.bindTooltip(stasiun.nama, {
                permanent: false,
                direction: 'top',
                offset: [0, -10]
            });

            return marker;
        }

        // Simpan marker ke array untuk fitur pencarian
        var markers = [];
        stasiunData.forEach(function(stasiun) {
            var m = createMarker(stasiun);
            markers.push({ marker: m, data: stasiun });
        });

        // =============================================
        // Modifikasi-3: Legenda Peta (pojok kanan bawah)
        // =============================================
        var legend = L.control({ position: 'bottomright' });

        legend.onAdd = function(map) {
            var div = L.DomUtil.create('div', 'legend');
            div.innerHTML =
                '<h4>🌧️ Tingkat Curah Hujan</h4>' +
                '<div class="legend-item"><span class="legend-dot" style="background:red;"></span> Tinggi (&gt; 2.500 mm/thn)</div>' +
                '<div class="legend-item"><span class="legend-dot" style="background:orange;"></span> Sedang (2.100 - 2.500 mm/thn)</div>' +
                '<div class="legend-item"><span class="legend-dot" style="background:green;"></span> Rendah (&lt; 2.100 mm/thn)</div>';
            return div;
        };

        legend.addTo(map);

        // =============================================
        // Modifikasi-5: Fitur Pencarian Stasiun Hujan (pojok kiri atas)
        // =============================================
        var SearchControl = L.Control.extend({
            options: { position: 'topleft' },
            onAdd: function(map) {
                var container = L.DomUtil.create('div', 'search-container');
                L.DomEvent.disableClickPropagation(container);

                var input = L.DomUtil.create('input', '', container);
                input.id = 'searchInput';
                input.type = 'text';
                input.placeholder = 'Cari stasiun hujan..';

                var btn = L.DomUtil.create('button', '', container);
                btn.id = 'searchBtn';
                btn.innerHTML = '🔍';

                // Fungsi pencarian
                function doSearch() {
                    var keyword = input.value.toLowerCase().trim();
                    var found = false;
                    markers.forEach(function(item) {
                        if (item.data.nama.toLowerCase().includes(keyword)) {
                            map.setView(item.data.koordinat, 15);
                            item.marker.openPopup();
                            found = true;
                        }
                    });
                    if (!found) {
                        alert('Stasiun hujan tidak ditemukan');
                    }
                }

                // Enter key
                L.DomEvent.on(input, 'keydown', function(e) {
                    if (e.key === 'Enter') doSearch();
                });

                // Klik tombol
                L.DomEvent.on(btn, 'click', doSearch);

                return container;
            }
        });

        map.addControl(new SearchControl());

        console.log('Peta Stasiun Hujan Kota Bandung berhasil dimuat!');
    </script>
</body>
</html><!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Peta Sebaran Stasiun Hujan Kota Bandung</title>
    
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <!-- Leaflet JS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    
    <style>
        #map {
            height: 500px;
            width: 100%;
            border: 1px solid #ccc;
            border-radius: 8px;
        }
        
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        
        h1 {
            color: #2c3e50;
            text-align: center;
        }
        
        .info {
            background-color: white;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 10px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        /* Modifikasi-3: Legenda */
        .legend {
            background: white;
            padding: 10px 14px;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
            font-size: 13px;
            line-height: 1.8;
            min-width: 200px;
        }

        .legend h4 {
            margin: 0 0 8px 0;
            font-size: 13px;
            color: #2c3e50;
            border-bottom: 1px solid #ddd;
            padding-bottom: 4px;
        }

        .legend-item {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 4px;
        }

        .legend-dot {
            width: 14px;
            height: 14px;
            border-radius: 50%;
            display: inline-block;
            border: 1px solid rgba(0,0,0,0.3);
            flex-shrink: 0;
        }

        /* Modifikasi-5: Search Box */
        .search-container {
            position: absolute;
            top: 10px;
            left: 60px;
            z-index: 1000;
            display: flex;
            gap: 4px;
        }

        #searchInput {
            padding: 7px 12px;
            border: 2px solid #3498db;
            border-radius: 6px;
            font-size: 13px;
            width: 220px;
            outline: none;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
        }

        #searchInput:focus {
            border-color: #2980b9;
            box-shadow: 0 2px 10px rgba(52,152,219,0.4);
        }

        #searchBtn {
            padding: 7px 12px;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-size: 13px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.15);
        }

        #searchBtn:hover {
            background: #2980b9;
        }
    </style>
</head>
<body>
    <!-- Modifikasi-1: Heading diubah -->
    <h1>🌧️ Peta Sebaran Stasiun Hujan & Curah Hujan Bandung</h1>
    <div class="info">
        <!-- Modifikasi-1: Teks info panel diubah -->
        <strong>ℹ️ Informasi:</strong> Klik pada marker stasiun hujan untuk melihat data curah hujan
    </div>
    <div id="map"></div>

    <script>
        // =============================================
        // Modifikasi-1: Koordinat awal = pusat Kota Bandung, zoom 13
        // =============================================
        var map = L.map('map').setView([-6.9175, 107.6191], 13);

        // =============================================
        // Modifikasi-4: Layer Control (Basemap)
        // =============================================
        var osmLayer = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            maxZoom: 19
        });

        var googleSatellite = L.tileLayer('https://mt1.google.com/vt/lyrs=s&x={x}&y={y}&z={z}', {
            attribution: '&copy; Google Maps',
            maxZoom: 20
        });

        var googleTerrain = L.tileLayer('https://mt1.google.com/vt/lyrs=p&x={x}&y={y}&z={z}', {
            attribution: '&copy; Google Maps',
            maxZoom: 20
        });

        // Tambahkan OSM sebagai default
        osmLayer.addTo(map);

        var baseMaps = {
            "OpenStreetMap": osmLayer,
            "Google Maps Satellite": googleSatellite,
            "Google Maps Terrain": googleTerrain
        };

        L.control.layers(baseMaps).addTo(map);

        // =============================================
        // Tambahkan scale bar
        // =============================================
        L.control.scale({
            metric: true,
            imperial: false
        }).addTo(map);

        // Pindahkan kontrol zoom ke kiri bawah
        map.zoomControl.setPosition('bottomleft');

        // =============================================
        // Modifikasi-2: Data 5 Stasiun Hujan Kota Bandung
        // =============================================
        var stasiunData = [
            {
                nama: "Stasiun Hujan Gedebage",
                koordinat: [-6.9439, 107.7082],
                ketinggian: 675,
                curahHujan: 2150
            },
            {
                nama: "Stasiun Hujan Dayeuhkolot",
                koordinat: [-6.9794, 107.6183],
                ketinggian: 660,
                curahHujan: 2080
            },
            {
                nama: "Stasiun Hujan Lembang",
                koordinat: [-6.8126, 107.6156],
                ketinggian: 1250,
                curahHujan: 2800
            },
            {
                nama: "Stasiun Hujan Cimahi",
                koordinat: [-6.8722, 107.5423],
                ketinggian: 740,
                curahHujan: 2200
            },
            {
                nama: "Stasiun Hujan Ujungberung",
                koordinat: [-6.9050, 107.6850],
                ketinggian: 685,
                curahHujan: 2100
            }
        ];

        // Fungsi warna marker berdasarkan curah hujan
        function getMarkerColor(curahHujan) {
            if (curahHujan > 2500) {
                return 'red';
            } else if (curahHujan >= 2100) {
                return 'orange';
            } else {
                return 'green';
            }
        }

        // Fungsi membuat circle marker
        function createMarker(stasiun) {
            var color = getMarkerColor(stasiun.curahHujan);

            var marker = L.circleMarker(stasiun.koordinat, {
                radius: 10,
                fillColor: color,
                color: '#333',
                weight: 1.5,
                opacity: 1,
                fillOpacity: 0.85
            }).addTo(map);

            // Popup berisi Nama Stasiun, Ketinggian, Curah Hujan Tahunan
            marker.bindPopup(
                '<b>📍 ' + stasiun.nama + '</b><br>' +
                '⛰️ Ketinggian: ' + stasiun.ketinggian + ' mdpl<br>' +
                '🌧️ Curah Hujan Tahunan: ' + stasiun.curahHujan + ' mm/thn'
            );

            // Tooltip muncul saat hover
            marker.bindTooltip(stasiun.nama, {
                permanent: false,
                direction: 'top',
                offset: [0, -10]
            });

            return marker;
        }

        // Simpan marker ke array untuk fitur pencarian
        var markers = [];
        stasiunData.forEach(function(stasiun) {
            var m = createMarker(stasiun);
            markers.push({ marker: m, data: stasiun });
        });

        // =============================================
        // Modifikasi-3: Legenda Peta (pojok kanan bawah)
        // =============================================
        var legend = L.control({ position: 'bottomright' });

        legend.onAdd = function(map) {
            var div = L.DomUtil.create('div', 'legend');
            div.innerHTML =
                '<h4>🌧️ Tingkat Curah Hujan</h4>' +
                '<div class="legend-item"><span class="legend-dot" style="background:red;"></span> Tinggi (&gt; 2.500 mm/thn)</div>' +
                '<div class="legend-item"><span class="legend-dot" style="background:orange;"></span> Sedang (2.100 - 2.500 mm/thn)</div>' +
                '<div class="legend-item"><span class="legend-dot" style="background:green;"></span> Rendah (&lt; 2.100 mm/thn)</div>';
            return div;
        };

        legend.addTo(map);

        // =============================================
        // Modifikasi-5: Fitur Pencarian Stasiun Hujan (pojok kiri atas)
        // =============================================
        var SearchControl = L.Control.extend({
            options: { position: 'topleft' },
            onAdd: function(map) {
                var container = L.DomUtil.create('div', 'search-container');
                L.DomEvent.disableClickPropagation(container);

                var input = L.DomUtil.create('input', '', container);
                input.id = 'searchInput';
                input.type = 'text';
                input.placeholder = 'Cari stasiun hujan..';

                var btn = L.DomUtil.create('button', '', container);
                btn.id = 'searchBtn';
                btn.innerHTML = '🔍';

                // Fungsi pencarian
                function doSearch() {
                    var keyword = input.value.toLowerCase().trim();
                    var found = false;
                    markers.forEach(function(item) {
                        if (item.data.nama.toLowerCase().includes(keyword)) {
                            map.setView(item.data.koordinat, 15);
                            item.marker.openPopup();
                            found = true;
                        }
                    });
                    if (!found) {
                        alert('Stasiun hujan tidak ditemukan');
                    }
                }

                // Enter key
                L.DomEvent.on(input, 'keydown', function(e) {
                    if (e.key === 'Enter') doSearch();
                });

                // Klik tombol
                L.DomEvent.on(btn, 'click', doSearch);

                return container;
            }
        });

        map.addControl(new SearchControl());

        console.log('Peta Stasiun Hujan Kota Bandung berhasil dimuat!');
    </script>
</body>
</html>
