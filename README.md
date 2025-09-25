# operational-dashboard-poc
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Operacional - PoC</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #1f77b4;
            --success: #2ca02c;
            --warning: #ff7f0e;
            --danger: #d62728;
            --gray: #f5f5f5;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background: #f8f9fa;
        }
        
        .dashboard-header {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            margin-bottom: 20px;
        }
        
        .sla-cards {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 20px;
        }
        
        .sla-card {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            border-left: 4px solid var(--primary);
        }
        
        .sla-card.critical {
            border-left-color: var(--danger);
        }
        
        .sla-card.warning {
            border-left-color: var(--warning);
        }
        
        .sla-card.healthy {
            border-left-color: var(--success);
        }
        
        .metric-value {
            font-size: 2em;
            font-weight: bold;
            margin: 10px 0;
        }
        
        .charts-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
        }
        
        .chart-card {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        
        .alert-badge {
            display: inline-block;
            padding: 5px 10px;
            border-radius: 20px;
            font-size: 0.8em;
            font-weight: bold;
        }
        
        .badge-critical { background: var(--danger); color: white; }
        .badge-warning { background: var(--warning); color: white; }
        .badge-healthy { background: var(--success); color: white; }
    </style>
</head>
<body>
    <div class="dashboard-header">
        <h1>🚀 Dashboard Operacional - PoC</h1>
        <p>Monitoreo en tiempo real de SLA y Performance</p>
        <div class="alert-badge badge-healthy">SISTEMA ESTABLE</div>
    </div>

    <div class="sla-cards">
        <div class="sla-card healthy">
            <h3>📈 Disponibilidad SLA</h3>
            <div class="metric-value">99.98%</div>
            <div>Objetivo: 99.9% ✅</div>
            <small>Última actualización: <span id="uptime-time">Hace 2 min</span></small>
        </div>
        
        <div class="sla-card healthy">
            <h3>⚡ Performance SLA</h3>
            <div class="metric-value">47.2 ms</div>
            <div>Objetivo: < 100ms ✅</div>
            <small>Latencia promedio</small>
        </div>
        
        <div class="sla-card warning">
            <h3>🔴 Error Rate</h3>
            <div class="metric-value">0.45%</div>
            <div>Objetivo: < 0.3% ⚠️</div>
            <small>Requiere atención</small>
        </div>
        
        <div class="sla-card healthy">
            <h3>👥 Throughput</h3>
            <div class="metric-value">1,247 RPS</div>
            <div>Capacidad: 2,000 RPS ✅</div>
            <small>Requests por segundo</small>
        </div>
    </div>

    <div class="charts-container">
        <div class="chart-card">
            <h3>Response Time Trend (Últimas 24h)</h3>
            <canvas id="responseTimeChart" height="200"></canvas>
        </div>
        
        <div class="chart-card">
            <h3>Error Rate vs SLA Threshold</h3>
            <canvas id="errorRateChart" height="200"></canvas>
        </div>
        
        <div class="chart-card">
            <h3>Uptime Monitoring (7 días)</h3>
            <canvas id="uptimeChart" height="200"></canvas>
        </div>
        
        <div class="chart-card">
            <h3>🔔 Alertas Activas</h3>
            <div id="alerts-container">
                <div class="alert-item">
                    <span class="alert-badge badge-warning">WARNING</span>
                    High response time en API Gateway
                    <small>Hace 15 min · Servicio: api-gateway</small>
                </div>
                <div class="alert-item">
                    <span class="alert-badge badge-critical">CRITICAL</span>
                    Database connection pool al 95%
                    <small>Hace 5 min · Servicio: database</small>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Datos mock para los gráficos
        const generateTimeData = (hours = 24) => {
            const now = new Date();
            return Array.from({length: hours}, (_, i) => {
                const time = new Date(now - i * 60 * 60 * 1000);
                return time.toLocaleTimeString('es-ES', {hour: '2-digit', minute: '2-digit'});
            }).reverse();
        };

        const mockData = {
            responseTimes: [45, 47, 52, 48, 43, 46, 49, 51, 55, 58, 52, 49, 47, 45, 44, 46, 48, 50, 53, 49, 47, 45, 44, 42],
            errorRates: [0.2, 0.3, 0.4, 0.5, 0.6, 0.5, 0.4, 0.3, 0.4, 0.5, 0.6, 0.7, 0.6, 0.5, 0.4, 0.3, 0.4, 0.5, 0.6, 0.7, 0.6, 0.5, 0.4, 0.3],
            uptime: [99.9, 99.8, 99.9, 100, 99.9, 99.8, 99.9, 100, 99.9, 99.8, 99.9, 100, 99.9, 99.8, 99.9, 100, 99.9, 99.8, 99.9, 100, 99.9, 99.8, 99.9, 100],
            slaThreshold: 0.3
        };

        // Inicializar gráficos
        const timeLabels = generateTimeData();

        // Gráfico 1: Response Time
        new Chart(document.getElementById('responseTimeChart'), {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [{
                    label: 'Response Time (ms)',
                    data: mockData.responseTimes,
                    borderColor: '#1f77b4',
                    backgroundColor: 'rgba(31, 119, 180, 0.1)',
                    tension: 0.4,
                    fill: true
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: { display: false }
                },
                scales: {
                    y: {
                        beginAtZero: false,
                        title: { display: true, text: 'Milisegundos' }
                    }
                }
            }
        });

        // Gráfico 2: Error Rate vs SLA
        new Chart(document.getElementById('errorRateChart'), {
            type: 'line',
            data: {
                labels: timeLabels,
                datasets: [
                    {
                        label: 'Error Rate (%)',
                        data: mockData.errorRates,
                        borderColor: '#d62728',
                        backgroundColor: 'rgba(214, 39, 40, 0.1)',
                        tension: 0.4,
                        fill: true
                    },
                    {
                        label: 'SLA Threshold',
                        data: Array(24).fill(mockData.slaThreshold),
                        borderColor: '#2ca02c',
                        borderDash: [5, 5],
                        backgroundColor: 'transparent',
                        pointRadius: 0
                    }
                ]
            },
            options: {
                responsive: true,
                scales: {
                    y: {
                        beginAtZero: true,
                        title: { display: true, text: 'Porcentaje' }
                    }
                }
            }
        });

        // Gráfico 3: Uptime
        new Chart(document.getElementById('uptimeChart'), {
            type: 'line',
            data: {
                labels: ['Lun', 'Mar', 'Mié', 'Jue', 'Vie', 'Sáb', 'Dom'],
                datasets: [{
                    label: 'Uptime (%)',
                    data: [99.9, 99.8, 100, 99.9, 99.7, 99.9, 100],
                    borderColor: '#2ca02c',
                    backgroundColor: 'rgba(44, 160, 44, 0.1)',
                    tension: 0.4,
                    fill: true
                }]
            },
            options: {
                responsive: true,
                plugins: {
                    legend: { display: false }
                },
                scales: {
                    y: {
                        min: 99.5,
                        max: 100.1,
                        title: { display: true, text: 'Porcentaje' }
                    }
                }
            }
        });

        // Simular actualización en tiempo real
        setInterval(() => {
            const now = new Date().toLocaleTimeString('es-ES');
            document.getElementById('uptime-time').textContent = `Última actualización: ${now}`;
            
            // Rotar estado de alertas para demo
            const badges = document.querySelectorAll('.alert-badge');
            badges.forEach(badge => {
                if (badge.classList.contains('badge-healthy')) {
                    badge.classList.replace('badge-healthy', 'badge-warning');
                    badge.textContent = 'SISTEMA CON ALERTAS';
                } else if (badge.classList.contains('badge-warning')) {
                    badge.classList.replace('badge-warning', 'badge-healthy');
                    badge.textContent = 'SISTEMA ESTABLE';
                }
            });
        }, 10000);
    </script>
</body>
</html>
