<template>
  <aside class="sidebar">
    <div>
      <h1><span>🎓</span> 论文智能体</h1>
      <small>基于 Intel OPEA 架构的云边协同写作助手</small>
    </div>

    <div class="sidebar-section">
      <h2>☁️ 云端集群 (Cloud)</h2>
      <div class="sb-badge">
        🟢 状态: 在线
      </div>
      <!-- <small> Cluster</small>

      <div class="metric">
        <div class="metric-label">RAG 检索延迟</div>
        <div class="metric-value">120 ms</div>
      </div> -->
    </div>

    <div class="sidebar-section">
      <h2>💻 边端环境 (Edge)</h2>
      <div class="sb-badge info">
        🔵 设备: Intel AI PC
      </div>
      <small>芯片: Intel Core Ultra 7</small><br />
      <small>模型: Deepseek R1 14b (INT4)</small>

      <div class="sb-progress-wrapper">
        <div class="sb-progress-label">
          <!-- <span>NPU 负载</span><span>40%</span> -->
        </div>
        <!-- <div class="sb-progress-bar">
          <div class="sb-progress-bar-inner"></div>
        </div> -->
      </div>
    </div>
  </aside>
</template>
