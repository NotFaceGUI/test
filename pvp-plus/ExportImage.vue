<template>
  <div v-if="visible" class="export-image-fullscreen">
    <!-- 手机端模式切换按钮 -->
    <div class="mobile-mode-toggle" v-if="isMobile">
      <button class="mode-toggle-btn" :class="{ active: mobileMode === 'edit' }" @click="mobileMode = 'edit'">
        编辑
      </button>
      <button class="mode-toggle-btn" :class="{ active: mobileMode === 'preview' }" @click="mobileMode = 'preview'">
        预览
      </button>
    </div>

    <!-- 左侧队伍选择区域 -->
    <div class="team-selection-panel" :class="{ 'mobile-hidden': isMobile && mobileMode === 'preview' }">
      <div class="panel-header">
        <h2>导出配置</h2>
      </div>


      <!-- 渲染样式配置 -->
      <div class="render-config-section">
        <h3 class="section-title">渲染样式</h3>
        <div class="render-mode-selector">
          <div class="render-mode-item" :class="{ active: renderConfig.mode === 'normal' }"
            @click="renderConfig.mode = 'normal'">
            <svg class="mode-icon" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-5 14H7v-2h7v2zm3-4H7v-2h10v2zm0-4H7V7h10v2z" />
            </svg>
            <span class="mode-name">普通模式</span>
          </div>
          <div class="render-mode-item" :class="{ active: renderConfig.mode === 'pvp' }"
            @click="renderConfig.mode = 'pvp'">
            <svg class="mode-icon" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M6.92 5H5l9 9 1-.94m4.96 6.06l-.84.84a.996.996 0 01-1.41 0l-3.12-3.12-2.68 2.66-1.41-1.41 2.68-2.68L9.98 12.2a.996.996 0 010-1.41l.84-.84c.39-.39 1.02-.39 1.41 0l2.68 2.68 2.68-2.68c.39-.39 1.02-.39 1.41 0l.84.84c.39.39.39 1.02 0 1.41l-2.68 2.68 2.68 2.68c.39.39.39 1.02 0 1.41z" />
            </svg>
            <span class="mode-name">PVP模式</span>
          </div>
        </div>

        <!-- PVP模式专用配置 -->
        <div v-if="renderConfig.mode === 'pvp'" class="pvp-config">
          <div class="config-item">
            <label class="config-label">
              <input type="checkbox" v-model="renderConfig.showEnemy" class="config-checkbox">
              <span class="checkbox-text">显示敌人队伍</span>
            </label>
          </div>
        </div>



        <!-- 布局配置 -->
        <div class="layout-config">
          <h4 class="config-subtitle">布局设置</h4>
          <div class="config-item">
            <label>队伍间距:</label>
            <div class="scale-selector">
              <input type="range" v-model.number="renderConfig.teamSpacing" min="-100" max="300" class="scale-range">
              <span class="scale-value">{{ renderConfig.teamSpacing }}px</span>
            </div>
          </div>
          <div class="config-item">
            <label>角色大小:</label>
            <div class="scale-selector">
              <input type="range" v-model.number="renderConfig.characterScale" min="0.5" max="2" step="0.1"
                class="scale-range">
              <span class="scale-value">{{ Math.round(renderConfig.characterScale * 100) }}%</span>
            </div>
          </div>
        </div>

        <!-- 高级设置区域 -->
        <div class="advanced-settings">
          <div class="advanced-header" @click="toggleAdvancedSettings">
            <h4 class="config-subtitle">高级设置</h4>
            <svg class="expand-icon" :class="{ expanded: showAdvancedSettings }" width="16" height="16"
              viewBox="0 0 24 24" fill="currentColor">
              <path d="M7.41 8.59L12 13.17l4.59-4.58L18 10l-6 6-6-6 1.41-1.41z" />
            </svg>
          </div>
          <div class="advanced-content" v-show="showAdvancedSettings">
            <div class="config-item">
              <label>导出质量:</label>
              <select v-model="advancedConfig.exportQuality" class="select-input">
                <option value="high">高质量</option>
                <option value="medium">中等质量</option>
                <option value="low">低质量</option>
              </select>
            </div>
            <div class="config-item">
              <label>图片压缩:</label>
              <div class="scale-selector">
                <input type="range" v-model.number="advancedConfig.compression" min="0" max="100" class="scale-range">
                <span class="scale-value">{{ advancedConfig.compression }}%</span>
              </div>
            </div>
            <div class="config-item">
              <label class="config-label">
                <input type="checkbox" v-model="advancedConfig.includeWatermark" class="config-checkbox">
                <span class="checkbox-text">添加水印</span>
              </label>
            </div>
            <div v-if="advancedConfig.includeWatermark" class="config-item">
              <label>水印文本:</label>
              <input type="text" v-model="advancedConfig.watermarkText" class="config-input" placeholder="请输入水印文本">
            </div>
            <div v-if="advancedConfig.includeWatermark" class="config-item">
              <label>水印位置:</label>
              <div class="watermark-position-controls">
                <div class="position-input-group">
                  <label>X:</label>
                  <input type="number" v-model.number="advancedConfig.watermarkX" class="position-input" min="0"
                    placeholder="X坐标">
                </div>
                <div class="position-input-group">
                  <label>Y:</label>
                  <input type="number" v-model.number="advancedConfig.watermarkY" class="position-input" min="0"
                    placeholder="Y坐标">
                </div>
              </div>
            </div>
            <div class="config-item">
              <label>样式选择:</label>
              <div class="style-selector">
                <div class="style-option">
                  <label class="style-label">
                    <input type="radio" v-model="advancedConfig.displayMode" value="avatar" class="style-radio">
                    <span class="style-text">头像模式</span>
                  </label>
                </div>
                <div class="style-option">
                  <label class="style-label">
                    <input type="radio" v-model="advancedConfig.displayMode" value="display" class="style-radio">
                    <span class="style-text">完整显示</span>
                  </label>
                </div>
              </div>
            </div>
            <div class="config-item">
              <label>边距设置:</label>
              <div class="scale-selector">
                <input type="range" v-model.number="advancedConfig.padding" min="0" max="50" class="scale-range">
                <span class="scale-value">{{ advancedConfig.padding }}px</span>
              </div>
            </div>
            <div class="config-item">
              <label>角色间距:</label>
              <div class="scale-selector">
                <input type="range" v-model.number="advancedConfig.characterGap" min="0" max="100" class="scale-range">
                <span class="scale-value">{{ advancedConfig.characterGap }}px</span>
              </div>
            </div>
            <div class="config-item">
              <label>容器样式:</label>
              <button class="btn btn-style-editor" @click="showStyleEditor = true">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                  <path
                    d="M12,15.5A3.5,3.5 0 0,1 8.5,12A3.5,3.5 0 0,1 12,8.5A3.5,3.5 0 0,1 15.5,12A3.5,3.5 0 0,1 12,15.5M19.43,12.97C19.47,12.65 19.5,12.33 19.5,12C19.5,11.67 19.47,11.34 19.43,11.03L21.54,9.37C21.73,9.22 21.78,8.95 21.66,8.73L19.66,5.27C19.54,5.05 19.27,4.96 19.05,5.05L16.56,6.05C16.04,5.66 15.5,5.32 14.87,5.07L14.5,2.42C14.46,2.18 14.25,2 14,2H10C9.75,2 9.54,2.18 9.5,2.42L9.13,5.07C8.5,5.32 7.96,5.66 7.44,6.05L4.95,5.05C4.73,4.96 4.46,5.05 4.34,5.27L2.34,8.73C2.22,8.95 2.27,9.22 2.46,9.37L4.57,11.03C4.53,11.34 4.5,11.67 4.5,12C4.5,12.33 4.53,12.65 4.57,12.97L2.46,14.63C2.27,14.78 2.22,15.05 2.34,15.27L4.34,18.73C4.46,18.95 4.73,19.03 4.95,18.95L7.44,17.94C7.96,18.34 8.5,18.68 9.13,18.93L9.5,21.58C9.54,21.82 9.75,22 10,22H14C14.25,22 14.46,21.82 14.5,21.58L14.87,18.93C15.5,18.68 16.04,18.34 16.56,17.94L19.05,18.95C19.27,19.03 19.54,18.95 19.66,18.73L21.66,15.27C21.78,15.05 21.73,14.78 21.54,14.63L19.43,12.97Z" />
                </svg>
                自定义样式
              </button>
            </div>
          </div>
        </div>

        <!-- 详细信息配置 -->
        <div class="advanced-settings">
          <div class="advanced-header" @click="toggleDetailSettings">
            <h4 class="config-subtitle">详细信息配置</h4>
            <svg class="expand-icon" :class="{ expanded: showDetailSettings }" width="16" height="16"
              viewBox="0 0 24 24" fill="currentColor">
              <path d="M7.41 8.59L12 13.17l4.59-4.58L18 10l-6 6-6-6 1.41-1.41z" />
            </svg>
          </div>
          <div class="advanced-content" v-show="showDetailSettings">
            <div class="config-item">
              <label class="config-label">
                <input type="checkbox" v-model="detailConfig.enabled" class="config-checkbox">
                <span class="checkbox-text">显示角色详细信息</span>
              </label>
            </div>

            <div v-if="detailConfig.enabled" class="detail-options">
              <!-- 基础信息选项 -->
              <div class="detail-category">
                <h4 class="detail-category-title">基础信息</h4>
                <div class="detail-items">
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showLevel" class="detail-checkbox">
                    <span class="detail-text">等级</span>
                  </label>
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showPower" class="detail-checkbox">
                    <span class="detail-text">战力</span>
                  </label>
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showCube" class="detail-checkbox">
                    <span class="detail-text">魔方</span>
                  </label>
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showFavoriteItem" class="detail-checkbox">
                    <span class="detail-text">收藏品</span>
                  </label>
                </div>
              </div>

              <!-- 装备信息选项 -->
              <div class="detail-category">
                <h4 class="detail-category-title">装备信息</h4>
                <div class="detail-items">
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showEquipment" class="detail-checkbox">
                    <span class="detail-text">装备等级</span>
                  </label>
                  <label class="detail-item-label">
                    <input type="checkbox" v-model="detailConfig.showEntries" class="detail-checkbox">
                    <span class="detail-text">装备词条</span>
                  </label>
                </div>
              </div>

              <!-- 自定义信息 -->
              <div class="detail-category">
                <h4 class="detail-category-title">自定义信息</h4>
                <div class="custom-info-list">
                  <div v-for="(customInfo, index) in detailConfig.customInfos" :key="index" class="custom-info-item">
                    <div class="custom-info-controls">
                      <input type="text" v-model="customInfo.label" placeholder="标签" class="custom-input custom-label">
                      <input type="text" v-model="customInfo.value" placeholder="内容" class="custom-input custom-value">
                      <input type="color" v-model="customInfo.color" class="custom-color">
                      <button @click="removeCustomInfo(index)" class="remove-btn">
                        <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                          <path
                            d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
                        </svg>
                      </button>
                    </div>
                    <div class="custom-info-image">
                      <label class="image-upload-label">
                        <input type="file" @change="handleCustomImageUpload($event, index)" accept="image/*"
                          class="image-upload-input">
                        <span class="image-upload-text">{{ customInfo.image ? '更换图片' : '上传图片' }}</span>
                      </label>
                      <div v-if="customInfo.image" class="image-preview">
                        <img :src="customInfo.image" alt="自定义图片" class="custom-image">
                        <button @click="removeCustomImage(index)" class="remove-image-btn">
                          <svg width="12" height="12" viewBox="0 0 24 24" fill="currentColor">
                            <path
                              d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
                          </svg>
                        </button>
                      </div>
                    </div>
                  </div>
                  <button @click="addCustomInfo" class="add-custom-btn">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                      <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z" />
                    </svg>
                    添加自定义信息
                  </button>
                </div>
              </div>

              <!-- 详细信息样式配置 -->
              <div class="detail-category">
                <h4 class="detail-category-title">样式配置</h4>
                <div class="detail-style-config">
                  <div class="config-item">
                    <label>信息位置:</label>
                    <select v-model="detailConfig.position" class="select-input">
                      <option value="bottom">底部</option>
                      <option value="right">右侧</option>
                      <option value="overlay">覆盖显示</option>
                    </select>
                  </div>
                  <div class="config-item">
                    <label>字体大小:</label>
                    <div class="scale-selector">
                      <input type="range" v-model.number="detailConfig.fontSize" min="10" max="20" class="scale-range">
                      <span class="scale-value">{{ detailConfig.fontSize }}px</span>
                    </div>
                  </div>
                  <div class="config-item">
                    <label>背景透明度:</label>
                    <div class="scale-selector">
                      <input type="range" v-model.number="detailConfig.backgroundOpacity" min="0" max="100"
                        class="scale-range">
                      <span class="scale-value">{{ detailConfig.backgroundOpacity }}%</span>
                    </div>
                  </div>
                  <div class="config-item">
                    <label>背景颜色:</label>
                    <input type="color" v-model="detailConfig.backgroundColor" class="color-input">
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>



      <!-- 队伍类型选择器 -->
      <div class="team-type-selector">
        <h3 class="section-title">选择队伍</h3>
        <div class="team-type-tabs">
          <div v-for="teamType in availableTeamTypes" :key="teamType.key" class="team-type-item"
            :class="{ active: selectedTeamType === teamType.key }" @click="selectedTeamType = teamType.key">
            <span class="team-type-icon" v-html="getTeamTypeIcon(teamType.key)"></span>
            <span class="team-type-name">{{ teamType.name }}</span>
          </div>
        </div>
      </div>

      <!-- 队伍列表 -->
      <div class="team-list">
        <div v-for="(team, index) in currentTeams" :key="`${selectedTeamType}-${index}`" class="team-item"
          :class="{ selected: selectedTeams.includes(`${selectedTeamType}-${index}`) }"
          @click="toggleTeam(`${selectedTeamType}-${index}`, index)">
          <div class="team-header">
            <h3 class="team-name">P{{ index + 1 }}</h3>
            <div class="checkbox" :class="{ checked: selectedTeams.includes(`${selectedTeamType}-${index}`) }">
              <span v-if="selectedTeams.includes(`${selectedTeamType}-${index}`)" style="font-size: 14px;">✓</span>
            </div>
          </div>
          <div class="team-preview">
            <div v-for="(character, charIndex) in team" :key="charIndex" class="character-slot">
              <div v-if="character" class="character-preview-wrapper">
                <CharacterAvatar :show-config-icon="false" :character-id="character"
                  :is-enemy="selectedTeamType === 'enemy'" />
              </div>
              <span v-else class="empty-slot">空</span>
            </div>
          </div>
        </div>
      </div>

      <!-- 操作栏 -->
      <div class="action-bar">
        <button class="btn btn-cancel" @click="handleCancel">取消</button>
        <button class="btn btn-export" :disabled="selectedTeams.length === 0" @click="handleExport">
          导出图片
        </button>
      </div>
    </div>

    <!-- 右侧预览画布区域 -->
    <div class="canvas-preview-panel" :class="{ 'mobile-hidden': isMobile && mobileMode === 'edit' }">
      <div class="canvas-header">
        <h2>预览画布</h2>
        <div class="canvas-controls">
          <button class="control-btn" @click="resetView" title="重置视图">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M12 4V1L8 5l4 4V6c3.31 0 6 2.69 6 6 0 1.01-.25 1.97-.7 2.8l1.46 1.46C19.54 15.03 20 13.57 20 12c0-4.42-3.58-8-8-8zm0 14c-3.31 0-6-2.69-6-6 0-1.01.25-1.97.7-2.8L5.24 7.74C4.46 8.97 4 10.43 4 12c0 4.42 3.58 8 8 8v3l4-4-4-4v3z" />
            </svg>
          </button>
          <span class="zoom-info">{{ Math.round(canvasScale * 100) }}%</span>
        </div>
      </div>

      <div ref="canvasContainer" class="canvas-container" @wheel="handleWheel" @mousedown="handleMouseDown"
        @mousemove="handleMouseMove" @mouseup="handleMouseUp" @mouseleave="handleMouseUp" @touchstart="handleTouchStart"
        @touchmove="handleTouchMove" @touchend="handleTouchEnd">
        <div class="canvas-content" :style="{
          transform: `translate(${canvasOffset.x}px, ${canvasOffset.y}px) scale(${canvasScale})`,
          transformOrigin: '0 0'
        }">
          <div v-if="selectedTeams.length === 0" class="no-selection">
            <div class="placeholder-icon">
              <svg width="48" height="48" viewBox="0 0 24 24" fill="currentColor">
                <path
                  d="M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-5 14H7v-2h7v2zm3-4H7v-2h10v2zm0-4H7V7h10v2z" />
              </svg>
            </div>
            <p>请从左侧选择要导出的队伍</p>
            <div class="config-preview">
              <p class="config-info">当前配置:</p>
              <p class="config-detail">模式: {{ renderConfig.mode === 'pvp' ? 'PVP模式' : '普通模式' }}</p>

              <p v-if="renderConfig.mode === 'pvp' && renderConfig.showEnemy" class="config-detail">✓ 显示敌人队伍</p>
            </div>
          </div>
          <div v-else class="teams-canvas" :style="{ position: 'relative', ...computedTeamsCanvasStyles }">
            <!-- PVP模式布局 -->
            <template v-if="renderConfig.mode === 'pvp'">
              <div class="pvp-layout">
                <!-- 我方队伍区域 -->
                <div class="ally-teams-section">
                  <h3 class="section-header">我方队伍</h3>
                  <div class="teams-grid">
                    <div v-for="(teamKey, index) in getAllyTeams" :key="teamKey" class="team-canvas-item pvp-team"
                      :style="{
                        ...computedTeamCanvasStyles,
                        marginBottom: `${renderConfig.teamSpacing}px`,
                        transform: `scale(${renderConfig.characterScale})`
                      }">
                      <div class="team-canvas-header" :style="computedHeaderStyles" :data-team-key="teamKey">
                        <h4 v-if="editingTitle !== teamKey" :style="{ ...computedTitleStyles, cursor: 'pointer' }"
                          @click="startEditTitle(teamKey)" :title="'点击编辑标题'">
                          {{ getPreviewTeamName(teamKey) }}
                        </h4>
                        <input v-else
                          :style="{ ...computedTitleStyles, background: 'transparent', border: '1px solid #007bff', borderRadius: '4px', padding: '2px 6px', outline: 'none' }"
                          :value="customTitles[teamKey] || getPreviewTeamName(teamKey)"
                          @keydown="handleTitleKeydown($event, teamKey)" @blur="handleTitleBlur($event, teamKey)"
                          maxlength="20" />
                      </div>
                      <div class="team-canvas-characters"
                        :style="{ ...computedCharactersStyles, gap: `${advancedConfig.characterGap}px` }">
                        <template v-if="advancedConfig.displayMode === 'avatar'">
                          <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                            class="character-canvas-slot">
                            <div v-if="character" class="character-canvas-wrapper">
                              <CharacterAvatar :show-config-icon="false" :character-id="character"
                                :is-enemy="teamKey && teamKey.startsWith('enemy')" />
                            </div>
                            <div v-else class="empty-canvas-slot">
                              <span>空</span>
                            </div>
                          </div>
                        </template>
                        <template v-else>
                          <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                            class="character-display-slot">
                            <div class="character-card">
                              <div class="character-mask">
                                <CharacterDisplay :character="getCharacterById(character)" />
                              </div>
                            </div>
                          </div>
                        </template>
                      </div>

                      <!-- 详细信息显示 -->
                      
                    </div>
                  </div>
                </div>

                <!-- 敌方队伍区域 -->
                <div v-if="renderConfig.showEnemy && getEnemyTeams.length > 0" class="enemy-teams-section">
                  <h3 class="section-header enemy-header">敌方队伍</h3>
                  <div class="teams-grid">
                    <div v-for="(teamKey, index) in getEnemyTeams" :key="teamKey"
                      class="team-canvas-item pvp-team enemy-team" :style="{
                        ...computedTeamCanvasStyles,
                        marginBottom: `${renderConfig.teamSpacing}px`,
                        transform: `scale(${renderConfig.characterScale})`
                      }">
                      <div class="team-canvas-header" :style="computedHeaderStyles">
                        <h4 :style="computedTitleStyles">{{ getPreviewTeamName(teamKey) }}</h4>
                      </div>
                      <div class="team-canvas-characters"
                        :style="{ ...computedCharactersStyles, gap: `${advancedConfig.characterGap}px` }">
                        <template v-if="advancedConfig.displayMode === 'avatar'">
                          <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                            class="character-canvas-slot">
                            <div v-if="character" class="character-canvas-wrapper">
                              <CharacterAvatar :show-config-icon="false" :character-id="character"
                                :is-enemy="teamKey && teamKey.startsWith('enemy')" />
                            </div>
                            <div v-else class="empty-canvas-slot">
                              <span>空</span>
                            </div>
                          </div>
                        </template>
                        <template v-else>
                          <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                            class="character-display-slot">
                            <div class="character-card">
                              <div class="character-mask">
                                <CharacterDisplay :character="getCharacterById(character)" />
                              </div>
                            </div>
                          </div>
                        </template>
                      </div>

                      <!-- 详细信息显示 -->

                    </div>
                  </div>
                </div>
              </div>
            </template>

            <!-- 普通模式布局 -->
            <template v-else>
              <div v-for="(teamKey, index) in selectedTeams" :key="teamKey" class="team-canvas-item" :style="{
                ...computedTeamCanvasStyles,
                marginTop: index > 0 ? `${renderConfig.teamSpacing}px` : '0',
                transform: `scale(${renderConfig.characterScale})`
              }">
                <div class="team-canvas-header" :style="computedHeaderStyles" :data-team-key="teamKey">
                  <h3 v-if="editingTitle !== teamKey" :style="{ ...computedTitleStyles, cursor: 'pointer' }"
                    @click="startEditTitle(teamKey)" :title="'点击编辑标题'">
                    {{ getPreviewTeamName(teamKey) }}
                  </h3>
                  <input v-else
                    :style="{ ...computedTitleStyles, background: 'transparent', border: '1px solid #007bff', borderRadius: '4px', padding: '2px 6px', outline: 'none' }"
                    :value="customTitles[teamKey] || getPreviewTeamName(teamKey)"
                    @keydown="handleTitleKeydown($event, teamKey)" @blur="handleTitleBlur($event, teamKey)"
                    maxlength="20" />
                </div>
                <div class="team-canvas-characters"
                  :style="{ ...computedCharactersStyles, gap: `${advancedConfig.characterGap}px` }">
                  <template v-if="advancedConfig.displayMode === 'avatar'">
                    <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                      class="character-canvas-slot">
                      <div v-if="character" class="character-canvas-wrapper">
                        <CharacterAvatar :show-config-icon="false" :character-id="character"
                          :is-enemy="teamKey && teamKey.startsWith('enemy')" />
                      </div>
                      <div v-else class="empty-canvas-slot">
                        <span>空</span>
                      </div>
                    </div>
                  </template>
                  <template v-else>
                    <div v-for="(character, charIndex) in getTeamData(teamKey)" :key="charIndex"
                      class="character-display-slot">
                      <div class="character-card">
                        <div class="character-mask">
                          <CharacterDisplay :character="getCharacterById(character)" />
                        </div>
                      </div>
                    </div>
                  </template>
                </div>

                <!-- 详细信息显示 -->
                <div v-if="detailConfig.enabled" class="team-detail-info" :style="computedDetailStyles">
                  <div v-for="(character, charIndex) in getTeamData(teamKey).filter(c => c)" :key="charIndex"
                    class="character-detail">
                    <div class="character-detail-header">
                      <span class="character-name">{{ getCharacterById(character)?.nikkeName || '未知角色' }}</span>
                    </div>
                    <div class="character-detail-content">
                      <!-- 基础信息 -->
                      <div class="detail-section" style="display: flex; gap: 20px; align-items: center;"
                        v-if="detailConfig.showLevel || detailConfig.showPower || (detailConfig.showCube && getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).selectedCube) || (detailConfig.showFavoriteItem && getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).favoriteItem)">
                        <div class="detail-row" v-if="detailConfig.showLevel" style="display: flex; align-items: center; gap: 5px;">
                          <span class="detail-label">等级:</span>
                          <span class="detail-value">{{ getCharacterConfig(character, teamKey &&
                            teamKey.startsWith('enemy')).level
                            }}</span>
                        </div>
                        <div class="detail-row" v-if="detailConfig.showPower" style="display: flex; align-items: center; gap: 5px;">
                          <span class="detail-label">战力:</span>
                          <span class="detail-value">{{ getCharacterConfig(character, teamKey &&
                            teamKey.startsWith('enemy')).power
                            }}</span>
                        </div>
                        <!-- 魔方信息 -->
                        <div class="cube-info" style="display: flex; align-items: center; gap: 5px;"
                          v-if="detailConfig.showCube && getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).selectedCube">
                          <span class="detail-label">魔方:</span>
                          <div class="cube-display" style="display: flex; align-items: center; gap: 3px;">
                            <img
                              :src="getCubeImagePath(getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).selectedCube)"
                              class="cube-image-small"
                              :alt="getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).cube" />
                            <span class="cube-level">Lv.{{ getCharacterConfig(character, teamKey &&
                              teamKey.startsWith('enemy')).cubeLevel || 1 }}</span>
                          </div>
                        </div>

                        <!-- 收藏品信息 -->
                        <div class="favorite-info" style="display: flex; align-items: center; gap: 5px;"
                          v-if="detailConfig.showFavoriteItem && getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).favoriteItem">
                          <span class="detail-label">收藏品:</span>
                          <div class="favorite-display" style="display: flex; align-items: center; gap: 3px;">
                            <img :src="getFavoriteItemImagePath(character, teamKey && teamKey.startsWith('enemy'))"
                              class="favorite-image-small" alt="收藏品" />
                            <span class="favorite-level">
                              Lv.{{ getCharacterConfig(character, teamKey &&
                                teamKey.startsWith('enemy')).favoriteItemLevel || 0 }}
                            </span>
                          </div>
                        </div>
                      </div>



                      <!-- 装备信息 -->
                      <div class="detail-section equipment-info" v-if="detailConfig.showEquipment">
                        <div class="detail-row">
                          <span class="detail-label">装备:</span>
                          <div class="equipment-grid">
                            <div
                              v-for="(tier, part) in getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')).equipment"
                              :key="part" class="equipment-item">
                              <div class="equipment-label">{{ getEquipmentName(part) }}</div>
                              <div class="equipment-card" :class="{ 't10': tier === 'T10' }">
                                <div v-if="tier != null" class="equipment-level">
                                  <div style="font-size: 12px;position: relative;bottom: -9px;opacity: 0.8;">LV</div>
                                  <div>
                                    0{{ getCharacterConfig(character, teamKey &&
                                      teamKey.startsWith('enemy')).equipmentLevel[part] || 1
                                    }}
                                  </div>
                                </div>
                                <img class="equipment-background" src="../assets/image/icn_character.png" alt="装备背景" />
                                <img v-if="tier" class="equipment-icon" :src="getEquipmentIcon(part, tier, character)"
                                  :alt="`${getEquipmentName(part)} ${tier}`" />
                                <div v-if="tier" class="equipment-tier-overlay" :class="tier?.toLowerCase()"></div>

                              </div>
                              <div style="width: 100%;margin-top: 5px;display: flex;justify-content: center;flex-direction: column;align-items: center;"
                                v-if="tier === 'T10' && detailConfig.showEntries">
                                <div class="equipment-ct" v-for="i in 3" :key="i"
                                  :style="getEntryContainerStyle(part, i - 1, getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')))">
                                  <template
                                    v-if="getCharacterConfig(character, teamKey && teamKey.startsWith('enemy'))?.entry && getCharacterConfig(character, teamKey && teamKey.startsWith('enemy'))?.entry[part]">
                                    <div style="font-size: 10px;"
                                      v-if="getCharacterConfig(character, teamKey && teamKey.startsWith('enemy'))?.entry[part][i - 1]">
                                      【{{ getCharacterConfig(character, teamKey &&
                                        teamKey.startsWith('enemy'))?.entry[part][i - 1].toString().split(":")[0]}}】</div>
                                    <div v-else
                                      style="font-size: 10px; opacity: 0.6;color: #333; text-align: center; width: 100%;">
                                      暂无词条
                                    </div>
                                    <div style="font-size: 10px;font-weight: bold;"
                                      :style="getEntryValueStyle(part, i - 1, getCharacterConfig(character, teamKey && teamKey.startsWith('enemy')))"
                                      v-if="getCharacterConfig(character, teamKey && teamKey.startsWith('enemy'))?.entry[part][i - 1]">
                                      {{ getCharacterConfig(character, teamKey &&
                                        teamKey.startsWith('enemy'))?.entry[part][i - 1].toString().split(":")[1]}}
                                    </div>
                                  </template>
                                  <div v-else>
                                    <div
                                      style="font-size: 10px; opacity: 0.6;color: #333; text-align: center; width: 100%;">
                                      暂无词条
                                    </div>
                                  </div>

                                </div>
                              </div>
                            </div>
                          </div>
                        </div>
                      </div>



                      <!-- 自定义信息 -->
                      <div class="detail-section custom-info"
                        v-if="detailConfig.customInfos && detailConfig.customInfos.length > 0">
                        <div v-for="(customInfo, customIndex) in detailConfig.customInfos" :key="customIndex"
                          class="detail-row">
                          <span class="detail-label" :style="{ color: customInfo.color }">{{ customInfo.label }}:</span>
                          <div class="custom-content">
                            <span class="detail-value" :style="{ color: customInfo.color }">{{ customInfo.value
                              }}</span>
                            <img v-if="customInfo.image" :src="customInfo.image" class="custom-info-image"
                              alt="自定义图片" />
                          </div>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </template>

            <!-- 水印层 -->
            <div v-if="advancedConfig.includeWatermark && advancedConfig.watermarkText" class="watermark-layer">
              <!-- 可拖动水印 -->
              <div class="watermark-draggable" :style="{
                left: advancedConfig.watermarkX + 'px',
                top: advancedConfig.watermarkY + 'px'
              }" @mousedown="startDragWatermark">
                <div class="watermark-item draggable-item">
                  {{ advancedConfig.watermarkText }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 导出选项弹窗 -->
    <div v-if="showExportDialog" class="export-dialog-overlay">
      <div class="export-dialog">
        <div class="dialog-header">
          <h3>导出设置</h3>
          <button class="close-btn" @click="cancelExport">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
            </svg>
          </button>
        </div>

        <div class="dialog-content">
          <div class="config-group">
            <label class="config-label">文件名:</label>
            <input type="text" v-model="exportConfig.filename" class="config-input" placeholder="请输入文件名">
          </div>

          <div class="config-group">
            <label class="config-label">格式:</label>
            <div class="format-selector">
              <label class="format-option">
                <input type="radio" v-model="exportConfig.format" value="png" class="format-radio">
                <span class="format-text">PNG (推荐)</span>
              </label>
              <label class="format-option">
                <input type="radio" v-model="exportConfig.format" value="jpeg" class="format-radio">
                <span class="format-text">JPEG</span>
              </label>
            </div>
          </div>

          <div class="config-group">
            <label class="config-label">分辨率倍数:</label>
            <div class="scale-selector">
              <input type="range" v-model.number="exportConfig.scale" min="1" max="4" step="0.5" class="scale-range">
              <span class="scale-value">{{ exportConfig.scale }}x</span>
            </div>
          </div>

          <div v-if="exportConfig.format === 'jpeg'" class="config-group">
            <label class="config-label">图片质量:</label>
            <div class="quality-selector">
              <input type="range" v-model.number="exportConfig.quality" min="0.1" max="1" step="0.1"
                class="quality-range">
              <span class="quality-value">{{ Math.round(exportConfig.quality * 100) }}%</span>
            </div>
          </div>

          <div class="config-group">
            <label class="config-label">背景设置:</label>
            <div class="background-selector">
              <div class="background-type-tabs">
                <label class="background-type-item">
                  <input type="radio" v-model="exportConfig.backgroundType" value="color" class="background-type-radio">
                  <span class="background-type-text">颜色背景</span>
                </label>
                <label class="background-type-item">
                  <input type="radio" v-model="exportConfig.backgroundType" value="transparent"
                    class="background-type-radio">
                  <span class="background-type-text">透明背景</span>
                </label>
              </div>
              <div v-if="exportConfig.backgroundType === 'color'" class="color-selector">
                <input type="color" v-model="exportConfig.backgroundColor" class="color-input">
                <span class="color-text">{{ exportConfig.backgroundColor }}</span>
              </div>

            </div>
          </div>
        </div>

        <div class="dialog-actions">
          <button class="btn btn-cancel" @click="cancelExport">取消</button>
          <button class="btn btn-export" @click="performExport">开始导出</button>
        </div>
      </div>
    </div>

    <!-- 导出进度弹窗 -->
    <div v-if="exportProgress.visible" class="progress-dialog-overlay">
      <div class="progress-dialog">
        <div class="progress-header">
          <h3>正在导出</h3>
        </div>

        <div class="progress-content">
          <div class="progress-spinner">
            <svg class="spinner" width="32" height="32" viewBox="0 0 24 24">
              <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round"
                stroke-dasharray="31.416" stroke-dashoffset="31.416">
                <animate attributeName="stroke-dasharray" dur="2s" values="0 31.416;15.708 15.708;0 31.416;0 31.416"
                  repeatCount="indefinite" />
                <animate attributeName="stroke-dashoffset" dur="2s" values="0;-15.708;-31.416;-31.416"
                  repeatCount="indefinite" />
              </circle>
            </svg>
          </div>
          <p class="progress-message">{{ exportProgress.message }}</p>
        </div>
      </div>
    </div>

    <!-- 样式编辑器弹窗 -->
    <div v-if="showStyleEditor" class="style-editor-overlay">
      <div class="style-editor-dialog">
        <div class="style-editor-header">
          <h3>容器样式编辑器</h3>
          <button class="close-btn" @click="showStyleEditor = false">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
              <path
                d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z" />
            </svg>
          </button>
        </div>
        <div class="style-editor-content">
          <StyleEditor ref="styleEditorRef" v-model="teamCanvasStyles" />
        </div>
        <div class="style-editor-actions">
          <button class="btn btn-cancel" @click="showStyleEditor = false">关闭</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, nextTick, onMounted, onUnmounted } from 'vue'
import { entryDictionary, useAppStore } from '../stores/app'
import CharacterAvatar from './CharacterAvatar.vue'
import CharacterDisplay from './CharacterDisplay.vue'
import StyleEditor from './StyleEditor.vue'
import { imageExporter, ExportStage } from '../utils/imageExport'
import { CharacterNameMap } from '../type/characters'
import { Profession } from '../type/var'


// Props
const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  }
})

// Emits
const emit = defineEmits(['update:visible', 'export'])

// Pinia store
const appStore = useAppStore()

// 响应式数据
const selectedTeams = ref([])
const selectedTeamType = ref('ally')

// 手机端检测和模式切换
const isMobile = ref(window.innerWidth <= 768)
const mobileMode = ref('edit') // 'edit' | 'preview'

// 监听窗口大小变化
const handleResize = () => {
  isMobile.value = window.innerWidth <= 768
  if (!isMobile.value) {
    mobileMode.value = 'edit' // 桌面端重置为编辑模式
  }
}

// 添加窗口大小监听
window.addEventListener('resize', handleResize)

// 渲染配置
const renderConfig = ref({
  mode: 'normal', // 'normal' | 'pvp'
  showEnemy: false, // PVP模式下是否显示敌人
  teamSpacing: 20, // 队伍间距
  characterScale: 1 // 角色缩放比例
})

// Canvas相关响应式数据
const canvasScale = ref(1)
const canvasOffset = ref({ x: 0, y: 0 })
const isDragging = ref(false)
const lastMousePos = ref({ x: 0, y: 0 })
const canvasContainer = ref(null)

// 触摸相关响应式数据
const isTouching = ref(false)
const lastTouchPos = ref({ x: 0, y: 0 })
const lastTouchDistance = ref(0)
const isMultiTouch = ref(false)

// 导出相关响应式数据
const showExportDialog = ref(false)
const exportProgress = ref({
  visible: false,
  stage: ExportStage.PREPARING,
  message: '准备中...'
})
const exportConfig = ref({
  format: 'png',
  scale: 2,
  quality: 0.92,
  backgroundType: 'color', // 'color' | 'transparent'
  backgroundColor: '#ffffff',
  filename: 'nikke-teams'
})

// 高级设置相关响应式数据
const showAdvancedSettings = ref(false)
const advancedConfig = ref({
  exportQuality: 'high', // 'high' | 'medium' | 'low'
  compression: 85, // 0-100
  includeWatermark: false,
  watermarkText: 'NIKKE PVP+', // 水印文本
  watermarkX: 20, // 水印X坐标
  watermarkY: 20, // 水印Y坐标
  displayMode: 'avatar', // 'avatar' | 'display'
  showAvatarStyle: true,
  padding: 20, // 0-50px
  characterGap: 12 // 0-30px
})

// 标题编辑相关响应式数据
const editingTitle = ref(null) // 当前正在编辑的标题key
const customTitles = ref({}) // 自定义标题存储

// 样式编辑器相关响应式数据
const showStyleEditor = ref(false)
const styleEditorRef = ref(null)
const teamCanvasStyles = ref({
  borderWidth: 2,
  borderStyle: 'solid',
  borderColor: '#e0e0e0',
  borderRadius: 12,
  backgroundColor: '#ffffff',
  backgroundOpacity: 1,
  shadowEnabled: true,
  shadowX: 0,
  shadowY: 4,
  shadowBlur: 12,
  shadowSpread: 0,
  shadowColor: '#000000',
  shadowOpacity: 0.1,
  padding: 20,
  // Teams Canvas背景设置
  teamsCanvasBackgroundType: 'none',
  teamsCanvasBackgroundColor: '#ffffff',
  teamsCanvasBackgroundOpacity: 1,
  teamsCanvasGradientDirection: '135deg',
  teamsCanvasGradientStart: '#ffffff',
  teamsCanvasGradientEnd: '#f0f0f0',
  teamsCanvasBackgroundImage: '',
  teamsCanvasBackgroundSize: 'cover',
  teamsCanvasBackgroundRepeat: 'no-repeat'
})

// 详细信息配置
const showDetailSettings = ref(false)
const detailConfig = ref({
  enabled: false,
  showLevel: true,
  showPower: true,
  showCube: true,
  showFavoriteItem: true,
  showEquipment: true,
  showEntries: true,
  customInfos: [],
  position: 'bottom',
  fontSize: 14,
  backgroundOpacity: 80,
  backgroundColor: '#fff'
})

// 监听渲染模式变化，自动调整选中的队伍类型
watch(() => renderConfig.value.mode, (newMode) => {
  // 当切换到PVP模式时，如果当前选中的是敌人队伍但未开启显示敌人，则切换到我方队伍
  if (newMode === 'pvp' && selectedTeamType.value === 'enemy' && !renderConfig.value.showEnemy) {
    selectedTeamType.value = 'ally'
  }
  // 清空已选择的队伍，避免模式切换时的数据混乱
  selectedTeams.value = []
})



// 监听显示敌人设置变化
watch(() => renderConfig.value.showEnemy, (showEnemy) => {
  if (!showEnemy && selectedTeamType.value === 'enemy') {
    selectedTeamType.value = 'ally'
  }
  // 如果关闭显示敌人，移除已选择的敌人队伍
  if (!showEnemy) {
    selectedTeams.value = selectedTeams.value.filter(teamKey => {
      const [teamType] = teamKey.split('-')
      return teamType !== 'enemy'
    })
  }
})

// 监听选中队伍变化，自动重新居中
watch(() => selectedTeams.value.length, () => {
  nextTick(() => {
    canvasOffset.value = getCenterOffset()
  })
}, { flush: 'post' })

// 监听渲染配置变化，自动重新居中
watch(() => renderConfig.value.mode, () => {
  nextTick(() => {
    canvasOffset.value = getCenterOffset()
  })
}, { flush: 'post' })

// 组件挂载后自动居中
onMounted(() => {
  nextTick(() => {
    canvasOffset.value = getCenterOffset()
  })
})

// 计算属性
const visible = computed({
  get: () => props.visible,
  set: (value) => emit('update:visible', value)
})

// 获取队伍类型图标SVG
const getTeamTypeIcon = (key) => {
  const icons = {
    ally: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2l3.09 6.26L22 9.27l-5 4.87 1.18 6.88L12 17.77l-6.18 3.25L7 14.14 2 9.27l6.91-1.01L12 2z"/></svg>',
    defense: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12,1L3,5V11C3,16.55 6.84,21.74 12,23C17.16,21.74 21,16.55 21,11V5L12,1M12,7C13.4,7 14.8,8.6 14.8,10V11H16V18H8V11H9.2V10C9.2,8.6 10.6,7 12,7M12,8.2C11.2,8.2 10.4,8.7 10.4,10V11H13.6V10C13.6,8.7 12.8,8.2 12,8.2Z"/></svg>',
    arena: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/></svg>',
    enemy: '<svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor"><path d="M12,1L3,5V11C3,16.55 6.84,21.74 12,23C17.16,21.74 21,16.55 21,11V5L12,1M11,7H13V9H11V7M11,11H13V17H11V11Z"/></svg>'
  }
  return icons[key] || icons.ally
}

// 根据渲染模式获取可用的队伍类型
const availableTeamTypes = computed(() => {
  if (renderConfig.value.mode === 'pvp') {
    const types = [
      { key: 'ally', name: '我方队伍' },
      { key: 'defense', name: '防御队伍' },
      { key: 'arena', name: '攻击队伍' }
    ]

    if (renderConfig.value.showEnemy) {
      types.push({ key: 'enemy', name: '敌方队伍' })
    }

    return types
  } else {
    return [
      { key: 'ally', name: '正常模式' },
      { key: 'defense', name: '防御队伍' },
      { key: 'arena', name: '攻击队伍' }
    ]
  }
})



// 获取当前选中类型的队伍数据
const currentTeams = computed(() => {
  const teams = []
  for (let i = 0; i < 5; i++) {
    switch (selectedTeamType.value) {
      case 'ally':
        teams.push(appStore.getAllyTeam(i))
        break
      case 'enemy':
        teams.push(appStore.getEnemyTeam(i))
        break
      case 'defense':
        teams.push(appStore.getDefenseTeam(i))
        break
      case 'arena':
        teams.push(appStore.getArenaTeam(i))
        break
      default:
        teams.push([null, null, null, null, null])
    }
  }
  return teams
})

// 获取我方队伍列表（用于PVP模式）
const getAllyTeams = computed(() => {
  return selectedTeams.value.filter(teamKey => {
    if (!teamKey || typeof teamKey !== 'string') return false
    const [teamType] = teamKey.split('-')
    return ['ally', 'defense', 'arena'].includes(teamType)
  })
})

// 获取敌方队伍列表（用于PVP模式）
const getEnemyTeams = computed(() => {
  return selectedTeams.value.filter(teamKey => {
    if (!teamKey || typeof teamKey !== 'string') return false
    const [teamType] = teamKey.split('-')
    return teamType === 'enemy'
  })
})

// 计算team-canvas-item的动态样式
const computedTeamCanvasStyles = computed(() => {
  const hexToRgba = (hex, opacity) => {
    // 处理rgba格式的颜色
    if (hex && hex.startsWith('rgba')) {
      return hex
    }
    const r = parseInt(hex.slice(1, 3), 16)
    const g = parseInt(hex.slice(3, 5), 16)
    const b = parseInt(hex.slice(5, 7), 16)
    return `rgba(${r}, ${g}, ${b}, ${opacity})`
  }

  // 解析自定义CSS
  const parseCustomCSS = (cssText) => {
    const styles = {}

    // 移除注释
    cssText = cssText.replace(/\/\*[\s\S]*?\*\//g, '')

    // 分割CSS属性
    const declarations = cssText.split(';').filter(decl => decl.trim())

    declarations.forEach(declaration => {
      const colonIndex = declaration.indexOf(':')
      if (colonIndex > 0) {
        const property = declaration.slice(0, colonIndex).trim()
        const value = declaration.slice(colonIndex + 1).trim()

        if (property && value) {
          // 转换CSS属性名为驼峰命名
          const camelCaseProperty = property.replace(/-([a-z])/g, (match, letter) => letter.toUpperCase())
          styles[camelCaseProperty] = value
        }
      }
    })

    return styles
  }

  const styles = {
    borderWidth: `${teamCanvasStyles.value.borderWidth}px`,
    borderStyle: teamCanvasStyles.value.borderStyle,
    borderColor: teamCanvasStyles.value.borderColor,
    borderRadius: `${teamCanvasStyles.value.borderRadius}px`,
    padding: `${teamCanvasStyles.value.padding}px`,
    transition: 'all 0.2s ease'
  }

  // 背景处理
  const backgroundType = teamCanvasStyles.value.backgroundType || 'color'
  if (backgroundType === 'color') {
    styles.backgroundColor = hexToRgba(teamCanvasStyles.value.backgroundColor, teamCanvasStyles.value.backgroundOpacity)
    styles.backgroundImage = 'none'
  } else if (backgroundType === 'gradient') {
    const startColor = hexToRgba(teamCanvasStyles.value.gradientStart, teamCanvasStyles.value.backgroundOpacity)
    const endColor = hexToRgba(teamCanvasStyles.value.gradientEnd, teamCanvasStyles.value.backgroundOpacity)
    const direction = teamCanvasStyles.value.gradientDirection || 'to right'
    styles.backgroundImage = `linear-gradient(${direction}, ${startColor}, ${endColor})`
    styles.backgroundColor = 'transparent'
  } else if (backgroundType === 'image' && teamCanvasStyles.value.backgroundImage) {
    styles.backgroundColor = hexToRgba(teamCanvasStyles.value.backgroundColor, teamCanvasStyles.value.backgroundOpacity)
    styles.backgroundImage = `url("${teamCanvasStyles.value.backgroundImage}")`
    styles.backgroundSize = teamCanvasStyles.value.backgroundSize || 'auto'
    styles.backgroundRepeat = teamCanvasStyles.value.backgroundRepeat || 'repeat'
    styles.backgroundPosition = 'center'
  } else {
    styles.backgroundColor = hexToRgba(teamCanvasStyles.value.backgroundColor, teamCanvasStyles.value.backgroundOpacity)
    styles.backgroundImage = 'none'
  }

  if (teamCanvasStyles.value.shadowEnabled) {
    const shadowColor = hexToRgba(teamCanvasStyles.value.shadowColor, teamCanvasStyles.value.shadowOpacity)
    styles.boxShadow = `${teamCanvasStyles.value.shadowX}px ${teamCanvasStyles.value.shadowY}px ${teamCanvasStyles.value.shadowBlur}px ${teamCanvasStyles.value.shadowSpread}px ${shadowColor}`
  } else {
    styles.boxShadow = 'none'
  }

  // 应用自定义CSS
  if (teamCanvasStyles.value.enableCustomCSS && teamCanvasStyles.value.customCSS) {
    try {
      // 解析自定义CSS并应用
      const customStyles = parseCustomCSS(teamCanvasStyles.value.customCSS)
      Object.assign(styles, customStyles)
    } catch (error) {
      console.warn('自定义CSS解析错误:', error)
    }
  }

  return styles
})

// 计算标题样式
const computedTitleStyles = computed(() => {
  return {
    fontSize: `${teamCanvasStyles.value.titleFontSize || 16}px`,
    fontWeight: teamCanvasStyles.value.titleFontWeight || '600',
    color: teamCanvasStyles.value.titleColor || '#333333',
    fontFamily: teamCanvasStyles.value.fontFamily || 'system-ui, -apple-system, sans-serif',
    textAlign: teamCanvasStyles.value.textAlign || 'left',
    margin: '0'
  }
})

// 计算头部样式
const computedHeaderStyles = computed(() => {
  return {
    marginBottom: `${teamCanvasStyles.value.headerSpacing || 12}px`,
    paddingBottom: (teamCanvasStyles.value.showHeaderBorder !== false) ? '8px' : '0',
    borderBottom: (teamCanvasStyles.value.showHeaderBorder !== false) ? `1px solid ${teamCanvasStyles.value.headerBorderColor || '#e0e0e0'}` : 'none'
  }
})

// 计算角色区域样式
const computedCharactersStyles = computed(() => {
  return {
    gap: `${teamCanvasStyles.value.characterSpacing || 8}px`
  }
})

// 计算Teams Canvas背景样式
const computedTeamsCanvasStyles = computed(() => {
  const hexToRgba = (hex, opacity) => {
    // 处理rgba格式的颜色
    if (hex && hex.startsWith('rgba')) {
      return hex
    }
    const r = parseInt(hex.slice(1, 3), 16)
    const g = parseInt(hex.slice(3, 5), 16)
    const b = parseInt(hex.slice(5, 7), 16)
    return `rgba(${r}, ${g}, ${b}, ${opacity})`
  }

  const result = {}

  // Teams Canvas背景处理
  if (teamCanvasStyles.value.teamsCanvasBackgroundType === 'color') {
    result.backgroundColor = hexToRgba(teamCanvasStyles.value.teamsCanvasBackgroundColor, teamCanvasStyles.value.teamsCanvasBackgroundOpacity)
    result.backgroundImage = 'none'
  } else if (teamCanvasStyles.value.teamsCanvasBackgroundType === 'gradient') {
    const startColor = hexToRgba(teamCanvasStyles.value.teamsCanvasGradientStart, teamCanvasStyles.value.teamsCanvasBackgroundOpacity)
    const endColor = hexToRgba(teamCanvasStyles.value.teamsCanvasGradientEnd, teamCanvasStyles.value.teamsCanvasBackgroundOpacity)
    result.backgroundImage = `linear-gradient(${teamCanvasStyles.value.teamsCanvasGradientDirection}, ${startColor}, ${endColor})`
    result.backgroundColor = 'transparent'
  } else if (teamCanvasStyles.value.teamsCanvasBackgroundType === 'image' && teamCanvasStyles.value.teamsCanvasBackgroundImage) {
    result.backgroundColor = hexToRgba(teamCanvasStyles.value.teamsCanvasBackgroundColor, teamCanvasStyles.value.teamsCanvasBackgroundOpacity)
    result.backgroundImage = `url("${teamCanvasStyles.value.teamsCanvasBackgroundImage}")`
    result.backgroundSize = teamCanvasStyles.value.teamsCanvasBackgroundSize
    result.backgroundRepeat = teamCanvasStyles.value.teamsCanvasBackgroundRepeat
    result.backgroundPosition = 'center'
  }
  // 如果是 'none' 类型，不设置任何背景样式

  return result
})

// 计算详细信息样式
const computedDetailStyles = computed(() => {
  const hexToRgba = (hex, opacity) => {
    if (hex && hex.startsWith('rgba')) {
      return hex
    }
    const r = parseInt(hex.slice(1, 3), 16)
    const g = parseInt(hex.slice(3, 5), 16)
    const b = parseInt(hex.slice(5, 7), 16)
    return `rgba(${r}, ${g}, ${b}, ${opacity / 100})`
  }

  return {
    fontSize: `${detailConfig.value.fontSize}px`,
    backgroundColor: hexToRgba(detailConfig.value.backgroundColor, detailConfig.value.backgroundOpacity)
  }
})

// 监听StyleEditor的样式变化，同步到本地teamCanvasStyles
watch(() => styleEditorRef.value?.computedTeamsCanvasStyles, (newStyles) => {
  if (newStyles && Object.keys(newStyles).length > 0) {
    // 从StyleEditor的样式中提取Teams Canvas相关的样式并保存到本地
    // 这样即使StyleEditor组件被销毁，样式也能保持
    const styleEditorStyles = styleEditorRef.value?.styles || {}
    if (styleEditorStyles.teamsCanvasBackgroundType !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundType = styleEditorStyles.teamsCanvasBackgroundType
    }
    if (styleEditorStyles.teamsCanvasBackgroundColor !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundColor = styleEditorStyles.teamsCanvasBackgroundColor
    }
    if (styleEditorStyles.teamsCanvasBackgroundOpacity !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundOpacity = styleEditorStyles.teamsCanvasBackgroundOpacity
    }
    if (styleEditorStyles.teamsCanvasGradientDirection !== undefined) {
      teamCanvasStyles.value.teamsCanvasGradientDirection = styleEditorStyles.teamsCanvasGradientDirection
    }
    if (styleEditorStyles.teamsCanvasGradientStart !== undefined) {
      teamCanvasStyles.value.teamsCanvasGradientStart = styleEditorStyles.teamsCanvasGradientStart
    }
    if (styleEditorStyles.teamsCanvasGradientEnd !== undefined) {
      teamCanvasStyles.value.teamsCanvasGradientEnd = styleEditorStyles.teamsCanvasGradientEnd
    }
    if (styleEditorStyles.teamsCanvasBackgroundImage !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundImage = styleEditorStyles.teamsCanvasBackgroundImage
    }
    if (styleEditorStyles.teamsCanvasBackgroundSize !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundSize = styleEditorStyles.teamsCanvasBackgroundSize
    }
    if (styleEditorStyles.teamsCanvasBackgroundRepeat !== undefined) {
      teamCanvasStyles.value.teamsCanvasBackgroundRepeat = styleEditorStyles.teamsCanvasBackgroundRepeat
    }
  }
}, { deep: true, immediate: true })

// 方法
// 根据角色ID获取完整的角色对象
const getCharacterById = (characterId) => {
  return CharacterNameMap[characterId] || null
}

const toggleTeam = (teamKey, index) => {
  const teamIndex = selectedTeams.value.indexOf(teamKey)
  if (teamIndex > -1) {
    selectedTeams.value.splice(teamIndex, 1)
  } else {
    selectedTeams.value.push(teamKey)
  }
}

// 切换高级设置显示
const toggleAdvancedSettings = () => {
  showAdvancedSettings.value = !showAdvancedSettings.value
}

// 手机端模式切换
const switchMobileMode = (mode) => {
  mobileMode.value = mode
}

const getTeamDisplayName = (teamType) => {
  const teamTypeObj = availableTeamTypes.value.find(t => t.key === teamType)
  return teamTypeObj ? teamTypeObj.name : '队伍'
}

const getPreviewTeamName = (teamKey) => {
  // 如果有自定义标题，优先使用自定义标题
  if (customTitles.value[teamKey]) {
    return customTitles.value[teamKey]
  }

  const [teamType, index] = teamKey.split('-')
  const teamTypeObj = availableTeamTypes.value.find(t => t.key === teamType)
  const typeName = teamTypeObj ? teamTypeObj.name : '队伍'
  return `${typeName} ${parseInt(index) + 1}`
}

const getTeamData = (teamKey) => {
  const [teamType, index] = teamKey.split('-')
  const teamIndex = parseInt(index)

  switch (teamType) {
    case 'ally':
      return appStore.getAllyTeam(teamIndex)
    case 'enemy':
      return appStore.getEnemyTeam(teamIndex)
    case 'defense':
      return appStore.getDefenseTeam(teamIndex)
    case 'arena':
      return appStore.getArenaTeam(teamIndex)
    default:
      return [null, null, null, null, null]
  }
}

// 标题编辑相关方法
const startEditTitle = (teamKey) => {
  editingTitle.value = teamKey
  // 延迟聚焦输入框
  nextTick(() => {
    const input = document.querySelector(`[data-team-key="${teamKey}"] input`)
    if (input) {
      input.focus()
      input.select()
    }
  })
}

const saveTitle = (teamKey, newTitle) => {
  if (newTitle && newTitle.trim()) {
    customTitles.value[teamKey] = newTitle.trim()
  } else {
    // 如果标题为空，删除自定义标题
    delete customTitles.value[teamKey]
  }
  editingTitle.value = null
}

const cancelEditTitle = () => {
  editingTitle.value = null
}

const handleTitleKeydown = (event, teamKey) => {
  if (event.key === 'Enter') {
    saveTitle(teamKey, event.target.value)
  } else if (event.key === 'Escape') {
    cancelEditTitle()
  }
}

const handleTitleBlur = (event, teamKey) => {
  saveTitle(teamKey, event.target.value)
}

const handleCancel = () => {
  visible.value = false
  selectedTeams.value = []
}

const handleExport = () => {
  if (selectedTeams.value.length === 0) return

  // 生成默认文件名
  const timestamp = new Date().toISOString().slice(0, 19).replace(/[:-]/g, '')
  const modePrefix = renderConfig.value.mode === 'pvp' ? 'pvp' : 'teams'
  exportConfig.value.filename = `nikke-${modePrefix}-${timestamp}`

  // 显示导出选项弹窗
  showExportDialog.value = true
}


// 获取词条数值的等级（在数组中的索引）
const getEntryValueLevel = (part, index, tempConfig) => {
  const partEntries = tempConfig.entry?.[part]
  if (partEntries && partEntries[index.toString()]) {
    const [entryName, value] = partEntries[index.toString()].split(':')
    if (entryName && value && entryDictionary[entryName]) {
      const valueIndex = entryDictionary[entryName].value.indexOf(value)
      return valueIndex >= 0 ? valueIndex : -1
    }
  }
  return -1
}


// 获取词条数值的样式类
const getEntryValueStyle = (part, index, tempConfig) => {
  const level = getEntryValueLevel(part, index, tempConfig)
  const partEntries = tempConfig.entry?.[part]

  if (level === -1 || !partEntries || !partEntries[index.toString()]) {
    return {}
  }

  const [entryName] = partEntries[index.toString()].split(':')
  if (!entryName || !entryDictionary[entryName]) {
    return {}
  }

  const maxLevel = entryDictionary[entryName].value.length - 1

  // 如果等级大于等于11（索引>=10）或者是最高等级
  if (level >= 10 || level === maxLevel) {
    if (level === maxLevel) {
      // 最高等级：蓝色文字
      return {
        color: '#007bff'
      }
    } else {
      // 11级以上：蓝色文字
      return {
        color: '#007bff'
      }
    }
  }

  return {}
}

// 获取词条容器的样式类
const getEntryContainerStyle = (part, index, tempConfig) => {
  const level = getEntryValueLevel(part, index, tempConfig)
  const partEntries = tempConfig.entry?.[part]


  if (level === -1 || !partEntries || !partEntries[index.toString()]) {
    return {}
  }

  const [entryName] = partEntries[index.toString()].split(':')
  if (!entryName || !entryDictionary[entryName]) {
    return {}
  }

  const maxLevel = entryDictionary[entryName].value.length - 1

  // 如果是最高等级，给整个容器添加黑色背景
  if (level === maxLevel) {
    return {
      backgroundColor: '#000000',
      borderRadius: '4px',
      color: "white",
      border: '1px solid #007bff'
    }
  }

  return {}
}

// 实际执行导出
const performExport = async () => {
  const canvasContent = canvasContainer.value?.querySelector('.canvas-content')
  if (!canvasContent) {
    console.error('找不到画布内容元素')
    return
  }

  // 关闭导出选项弹窗，显示进度
  showExportDialog.value = false
  exportProgress.value.visible = true

  // 获取实际的队伍内容元素
  const teamsCanvas = canvasContent.querySelector('.teams-canvas')
  if (!teamsCanvas) {
    console.error('找不到队伍内容元素')
    return
  }

  // 保存原始样式
  const originalTransform = canvasContent.style.transform
  const originalTransition = canvasContent.style.transition
  const originalTeamsTransform = teamsCanvas.style.transform

  try {
    // 临时移除 canvas-content 的 transform，但保持内容的缩放
    canvasContent.style.transform = 'none'
    canvasContent.style.transition = 'none'

    teamsCanvas.style.transform = `scale(${renderConfig.value.characterScale})`

    // 等待样式应用
    await new Promise(resolve => setTimeout(resolve, 100))

    await imageExporter.exportToImage({
      element: canvasContent,
      filename: exportConfig.value.filename,
      format: exportConfig.value.format,
      scale: exportConfig.value.scale,
      quality: exportConfig.value.quality,
      backgroundColor: exportConfig.value.backgroundType === 'transparent' ? null : exportConfig.value.backgroundColor,
      onProgress: (stage) => {
        exportProgress.value.stage = stage
        exportProgress.value.message = getProgressMessage(stage)
      },
      onSuccess: () => {
        setTimeout(() => {
          exportProgress.value.visible = false
          visible.value = false
          selectedTeams.value = []
        }, 1500)
      },
      onError: (error) => {
        console.error('导出失败:', error)
        exportProgress.value.message = `导出失败: ${error.message}`
        setTimeout(() => {
          exportProgress.value.visible = false
        }, 3000)
      }
    })
  } catch (error) {
    console.error('导出过程中发生错误:', error)
    exportProgress.value.message = `导出失败: ${error.message}`
    setTimeout(() => {
      exportProgress.value.visible = false
    }, 3000)
  } finally {
    // 恢复原始样式
    canvasContent.style.transform = originalTransform
    canvasContent.style.transition = originalTransition

    // 恢复队伍内容的原始样式
    if (teamsCanvas && typeof originalTeamsTransform !== 'undefined') {
      teamsCanvas.style.transform = originalTeamsTransform
    }
  }
}

// 获取进度消息
const getProgressMessage = (stage) => {
  const messages = {
    [ExportStage.PREPARING]: '准备导出...',
    [ExportStage.CLONING]: '复制元素...',
    [ExportStage.SCALING]: '应用缩放...',
    [ExportStage.RENDERING]: '渲染图片...',
    [ExportStage.DOWNLOADING]: '下载文件...',
    [ExportStage.COMPLETED]: '导出完成！',
    [ExportStage.ERROR]: '导出失败'
  }
  return messages[stage] || '处理中...'
}

// 取消导出
const cancelExport = () => {
  showExportDialog.value = false
}

// Canvas交互方法
const getCenterOffset = () => {
  const container = canvasContainer.value
  if (!container) return { x: 0, y: 0 }

  const containerRect = container.getBoundingClientRect()
  const containerWidth = containerRect.width
  const containerHeight = containerRect.height

  // 获取canvas内容元素
  const canvasContent = container.querySelector('.canvas-content')
  if (!canvasContent) return { x: 0, y: 0 }

  // 临时重置transform为只保留缩放，移除偏移
  const currentScale = canvasScale.value
  canvasContent.style.transform = `translate(0px, 0px) scale(${currentScale})`

  const contentRect = canvasContent.getBoundingClientRect()

  // 计算居中偏移（考虑缩放后的尺寸）
  const offsetX = (containerWidth - contentRect.width) / 2
  const offsetY = (containerHeight - contentRect.height) / 2

  return {
    x: offsetX,
    y: offsetY
  }
}

const resetView = () => {
  canvasScale.value = 1
  canvasOffset.value = getCenterOffset()
}

const handleWheel = (event) => {
  event.preventDefault()

  const container = canvasContainer.value
  if (!container) return

  const rect = container.getBoundingClientRect()
  const mouseX = event.clientX - rect.left
  const mouseY = event.clientY - rect.top

  const delta = event.deltaY > 0 ? 0.9 : 1.1
  const oldScale = canvasScale.value
  const newScale = Math.max(0.1, Math.min(3, oldScale * delta))

  if (newScale !== oldScale) {
    // 计算缩放中心点偏移
    const scaleRatio = newScale / oldScale - 1
    const offsetX = (mouseX - canvasOffset.value.x) * scaleRatio
    const offsetY = (mouseY - canvasOffset.value.y) * scaleRatio

    canvasScale.value = newScale
    canvasOffset.value = {
      x: canvasOffset.value.x - offsetX,
      y: canvasOffset.value.y - offsetY
    }
  }
}

const handleMouseDown = (event) => {
  isDragging.value = true
  lastMousePos.value = { x: event.clientX, y: event.clientY }

  // 拖拽时禁用过渡效果
  const canvasContent = document.querySelector('.canvas-content')
  if (canvasContent) {
    canvasContent.classList.add('dragging')
  }
}

const handleMouseMove = (event) => {
  if (!isDragging.value) return

  const deltaX = event.clientX - lastMousePos.value.x
  const deltaY = event.clientY - lastMousePos.value.y

  canvasOffset.value = {
    x: canvasOffset.value.x + deltaX,
    y: canvasOffset.value.y + deltaY
  }

  lastMousePos.value = { x: event.clientX, y: event.clientY }
}

const handleMouseUp = () => {
  isDragging.value = false

  // 拖拽结束时恢复过渡效果
  const canvasContent = document.querySelector('.canvas-content')
  if (canvasContent) {
    canvasContent.classList.remove('dragging')
  }
}

// 触摸事件处理
const getTouchDistance = (touches) => {
  if (touches.length < 2) return 0
  const touch1 = touches[0]
  const touch2 = touches[1]
  return Math.sqrt(
    Math.pow(touch2.clientX - touch1.clientX, 2) +
    Math.pow(touch2.clientY - touch1.clientY, 2)
  )
}

const getTouchCenter = (touches) => {
  if (touches.length === 1) {
    return { x: touches[0].clientX, y: touches[0].clientY }
  }
  const touch1 = touches[0]
  const touch2 = touches[1]
  return {
    x: (touch1.clientX + touch2.clientX) / 2,
    y: (touch1.clientY + touch2.clientY) / 2
  }
}

const handleTouchStart = (event) => {
  event.preventDefault()

  const touches = event.touches
  isTouching.value = true
  isMultiTouch.value = touches.length > 1

  if (touches.length === 1) {
    // 单指拖拽
    lastTouchPos.value = { x: touches[0].clientX, y: touches[0].clientY }

    // 拖拽时禁用过渡效果
    const canvasContent = document.querySelector('.canvas-content')
    if (canvasContent) {
      canvasContent.classList.add('dragging')
    }
  } else if (touches.length === 2) {
    // 双指缩放
    lastTouchDistance.value = getTouchDistance(touches)
    lastTouchPos.value = getTouchCenter(touches)
  }
}

const handleTouchMove = (event) => {
  event.preventDefault()

  if (!isTouching.value) return

  const touches = event.touches

  if (touches.length === 1 && !isMultiTouch.value) {
    // 单指拖拽
    const deltaX = touches[0].clientX - lastTouchPos.value.x
    const deltaY = touches[0].clientY - lastTouchPos.value.y

    canvasOffset.value = {
      x: canvasOffset.value.x + deltaX,
      y: canvasOffset.value.y + deltaY
    }

    lastTouchPos.value = { x: touches[0].clientX, y: touches[0].clientY }
  } else if (touches.length === 2) {
    // 双指缩放
    const currentDistance = getTouchDistance(touches)
    const currentCenter = getTouchCenter(touches)

    if (lastTouchDistance.value > 0) {
      const container = canvasContainer.value
      if (!container) return

      const rect = container.getBoundingClientRect()
      const centerX = currentCenter.x - rect.left
      const centerY = currentCenter.y - rect.top

      const scaleRatio = currentDistance / lastTouchDistance.value
      const oldScale = canvasScale.value
      const newScale = Math.max(0.1, Math.min(3, oldScale * scaleRatio))

      if (newScale !== oldScale) {
        // 计算缩放中心点偏移
        const scaleChange = newScale / oldScale - 1
        const offsetX = (centerX - canvasOffset.value.x) * scaleChange
        const offsetY = (centerY - canvasOffset.value.y) * scaleChange

        canvasScale.value = newScale
        canvasOffset.value = {
          x: canvasOffset.value.x - offsetX,
          y: canvasOffset.value.y - offsetY
        }
      }
    }

    lastTouchDistance.value = currentDistance
    lastTouchPos.value = currentCenter
  }
}

const handleTouchEnd = (event) => {
  event.preventDefault()

  const touches = event.touches

  if (touches.length === 0) {
    // 所有手指离开
    isTouching.value = false
    isMultiTouch.value = false
    lastTouchDistance.value = 0

    // 拖拽结束时恢复过渡效果
    const canvasContent = document.querySelector('.canvas-content')
    if (canvasContent) {
      canvasContent.classList.remove('dragging')
    }
  } else if (touches.length === 1 && isMultiTouch.value) {
    // 从双指变为单指
    isMultiTouch.value = false
    lastTouchDistance.value = 0
    lastTouchPos.value = { x: touches[0].clientX, y: touches[0].clientY }
  }
}

// 水印拖拽相关
const isDraggingWatermark = ref(false)
const watermarkDragStart = ref({ x: 0, y: 0 })

// 开始拖拽水印
const startDragWatermark = (event) => {
  event.stopPropagation() // 阻止事件冒泡到画布拖拽
  isDraggingWatermark.value = true

  const rect = canvasContainer.value.getBoundingClientRect()
  watermarkDragStart.value = {
    x: event.clientX - rect.left - advancedConfig.value.watermarkX,
    y: event.clientY - rect.top - advancedConfig.value.watermarkY
  }

  // 添加全局鼠标事件监听
  document.addEventListener('mousemove', handleWatermarkDrag)
  document.addEventListener('mouseup', stopDragWatermark)
}

// 拖拽水印过程
const handleWatermarkDrag = (event) => {
  if (!isDraggingWatermark.value) return

  const rect = canvasContainer.value.getBoundingClientRect()
  const newX = event.clientX - rect.left - watermarkDragStart.value.x
  const newY = event.clientY - rect.top - watermarkDragStart.value.y

  // 限制水印在画布范围内
  const canvasContent = canvasContainer.value.querySelector('.canvas-content')
  if (canvasContent) {
    const contentRect = canvasContent.getBoundingClientRect()
    const containerRect = canvasContainer.value.getBoundingClientRect()

    const maxX = (contentRect.width / canvasScale.value) - 100 // 减去水印宽度
    const maxY = (contentRect.height / canvasScale.value) - 30 // 减去水印高度

    advancedConfig.value.watermarkX = Math.max(0, Math.min(newX / canvasScale.value, maxX))
    advancedConfig.value.watermarkY = Math.max(0, Math.min(newY / canvasScale.value, maxY))
  }
}

// 停止拖拽水印
const stopDragWatermark = () => {
  isDraggingWatermark.value = false

  // 移除全局鼠标事件监听
  document.removeEventListener('mousemove', handleWatermarkDrag)
  document.removeEventListener('mouseup', stopDragWatermark)
}

// 详细信息配置相关方法
const toggleDetailSettings = () => {
  showDetailSettings.value = !showDetailSettings.value
}

const addCustomInfo = () => {
  detailConfig.value.customInfos.push({
    label: '',
    value: '',
    color: '#000000',
    image: null
  })
}

const removeCustomInfo = (index) => {
  detailConfig.value.customInfos.splice(index, 1)
}

const handleCustomImageUpload = (event, index) => {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      detailConfig.value.customInfos[index].image = e.target.result
    }
    reader.readAsDataURL(file)
  }
}

const removeCustomImage = (index) => {
  detailConfig.value.customInfos[index].image = null
}

// 获取角色配置信息
const getCharacterConfig = (characterId, isEnemy = false) => {
  return appStore.getCharacterConfig(characterId, isEnemy)
}

const getWeaponFavoriteItemPath = (weaponType, _levelr = 0) => {
  // 武器类型映射到英文缩写
  const weaponTypeMap = {
    '冲锋枪': 'smg',
    '突击步枪': 'ar',
    '发射器': 'rl',
    '霰弹枪': 'sg',
    '机枪': 'mg',
    '狙击步枪': 'sr'
  }
  const weaponCode = weaponTypeMap[weaponType] || 'ar'
  return getImagePath(`res/mi_favoriteitem_${weaponCode}_00.png`)
}

const getCharacterTreasurePath = (characterId, _tier = 1) => {
  return getImagePath(`res/mi_favoriteitem_${characterId}.png`)
}

// 获取装备名称
const getEquipmentName = (part) => {
  const names = {
    helmet: '头盔',
    armor: '护甲',
    boots: '靴子',
    weapon: '武器'
  }
  return names[part] || part
}

// 检查是否有T10装备
const hasT10Equipment = (characterId, isEnemy = false) => {
  const config = getCharacterConfig(characterId, isEnemy)
  if (!config || !config.equipment) return false

  return Object.values(config.equipment).some(tier => tier === 'T10')
}

const getImagePath = (path) => {
  let url = `../assets/${path}`;
  return new URL(url, import.meta.url).href;
}

// 获取魔方图片路径
const getCubeImagePath = (cubeId) => {
  return getImagePath(`res/${cubeId}.png`)
}

// 获取收藏品图片路径
const getFavoriteItemImagePath = (characterId, isEnemy = false) => {
  const config = getCharacterConfig(characterId, isEnemy)
  const character = getCharacterById(characterId)

  if (!character || !config.favoriteItem) return ''

  if (config.favoriteItem === 'weapon') {
    const level = config.favoriteItemLevel || 0
    const weaponTypeMap = {
      '冲锋枪': 'smg',
      '突击步枪': 'ar',
      '发射器': 'rl',
      '霰弹枪': 'sg',
      '机枪': 'mg',
      '狙击步枪': 'sr'
    }
    const weaponCode = weaponTypeMap[character.Weapon] || 'ar'
    return getImagePath(`res/mi_favoriteitem_${weaponCode}_00.png`)
  } else if (config.favoriteItem === 'character') {
    return getImagePath(`res/mi_favoriteitem_${characterId}.png`)
  }
  return ''
}

// 获取装备图标路径
const getEquipmentIcon = (part, tier, characterId) => {
  if (!tier) return ''

  const character = getCharacterById(characterId)
  // 如果角色数据不存在，使用默认的attacker职业
  const profession = character?.profession ? (
    character.profession === Profession.ATTACKER ? 'attacker' :
      character.profession === Profession.DEFENDER ? 'defender' :
        character.profession === Profession.SUPPORTER ? 'supporter' :
          'attacker'
  ) : 'attacker'


  const tierNum = tier === 'T9' ? '1' : '3'
  return getImagePath(`res/icn_equipment_${part}_${profession}_t9_${tierNum}.png`)
}



// 组件销毁时清理事件监听
onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  // 清理水印拖拽事件监听
  document.removeEventListener('mousemove', handleWatermarkDrag)
  document.removeEventListener('mouseup', stopDragWatermark)
})
</script>

<style scoped>
.export-image-fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: #f5f5f5;
  z-index: 1000;
  display: flex;
  overflow: hidden;
}

/* 左侧队伍选择面板 */
.team-selection-panel {
  width: 40%;
  background: #ffffff;
  border-right: 2px solid #e0e0e0;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
}

.team-selection-panel::-webkit-scrollbar {
  width: 8px;
}

.team-selection-panel::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 4px;
}

.team-selection-panel::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 4px;
}

.team-selection-panel::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}

.panel-header {
  padding: 20px;
  border-bottom: 1px solid #e0e0e0;
  background: #fafafa;
}

.panel-header h2 {
  margin: 0;
  color: #333;
  font-size: 20px;
  font-weight: 600;
  text-align: center;
}

.team-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  padding: 20px;
  gap: 20px;
  width: 100%;
  box-sizing: border-box;
  place-items: center;
  place-content: center;
}



/* 渲染配置样式 */
.render-config-section {
  margin-bottom: 30px;
  padding: 15px;
  background: #f8f9fa;
  border-radius: 8px;
  border: 1px solid #e9ecef;
}

.section-title {
  margin: 0 0 15px 0;
  font-size: 16px;
  font-weight: 600;
  color: #495057;
}

.config-subtitle {
  margin: 15px 0 10px 0;
  font-size: 14px;
  font-weight: 500;
  color: #6c757d;
}

.render-mode-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.render-mode-item {
  flex: 1;
  padding: 12px;
  background: white;
  border: 2px solid #dee2e6;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.render-mode-item:hover {
  border-color: #007bff;
  background: #f8f9fa;
}

.render-mode-item.active {
  border-color: #007bff;
  background: #e3f2fd;
  color: #0056b3;
}

.mode-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  margin-bottom: 5px;
  width: 20px;
  height: 20px;
}

.mode-name {
  font-size: 12px;
  font-weight: 500;
  line-height: 1.2;
}

.pvp-config {
  margin-top: 15px;
  padding: 10px;
  background: white;
  border-radius: 6px;
  border: 1px solid #dee2e6;
}

.config-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 10px;
}

.config-item:last-child {
  margin-bottom: 0;
}

.config-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 14px;
}

.config-checkbox {
  margin-right: 8px;
}

.checkbox-text {
  color: #495057;
}



.layout-config {
  margin-top: 15px;
  padding: 10px;
  background: white;
  border-radius: 6px;
  border: 1px solid #dee2e6;
}

.range-input {
  flex: 1;
  margin: 0 10px;
}

.range-value {
  font-size: 12px;
  color: #495057;
  min-width: 50px;
  text-align: right;
}

.config-item label {
  font-size: 12px;
  color: #6c757d;
  min-width: 60px;
}

.team-type-selector {
  margin-bottom: 20px;
  padding: 0 20px;
}

.team-type-tabs {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

/* 右侧canvas预览面板 */
.canvas-preview-panel {
  width: 60%;
  background: #f8f9fa;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.canvas-header {
  padding: 20px;
  border-bottom: 1px solid #e0e0e0;
  background: #ffffff;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.canvas-header h2 {
  margin: 0;
  color: #333;
  font-size: 20px;
  font-weight: 600;
}

.canvas-controls {
  display: flex;
  align-items: center;
  gap: 12px;
}

.control-btn {
  padding: 8px;
  background: #3E4042;
  border: none;
  border-radius: 6px;
  color: white;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.control-btn:hover {
  background: #16AEF3;
}

.zoom-info {
  font-size: 14px;
  color: #666;
  font-weight: 500;
  min-width: 50px;
  text-align: center;
}

.canvas-container {
  flex: 1;
  position: relative;
  overflow: hidden;
  cursor: grab;
  background: #ffffff;
  /* 点状背景，类似节点图 */
  background-image: radial-gradient(circle, #d0d0d0 1px, transparent 1px);
  background-size: 20px 20px;
  background-position: 0 0, 10px 10px;
  /* 禁用全局transition以确保拖拽流畅 */
  transition: none !important;
}

.canvas-container:active {
  cursor: grabbing;
}

.canvas-content {
  width: fit-content;
  position: relative;
  /* 为缩放添加平滑过渡，但拖拽时会被动态禁用 */
  transition: transform 0.2s ease-out;
}

.canvas-content.dragging {
  /* 拖拽时禁用过渡以确保流畅 */
  transition: none !important;
}

.canvas-content.dragging * {
  /* 拖拽时禁用所有子元素的过渡 */
  transition: none !important;
}


.team-type-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  background: #3E4042;
  border: 2px solid #3E4042;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  min-width: 140px;
  justify-content: center;
  color: white;
}

.team-type-item:hover {
  border-color: #16AEF3;
  background: #2a2c2e;
}

.team-type-item.active {
  border-color: #16AEF3;
  background: #16AEF3;
  color: white;
}

.team-type-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  font-size: 18px;
}

.team-type-name {
  font-size: 14px;
  font-weight: 500;
  line-height: 1.2;
  white-space: nowrap;
}

.team-type-name {
  font-size: 14px;
  font-weight: 500;
}

.team-list .team-item {
  width: fit-content;
  height: fit-content;
  margin-bottom: 12px;
}

.team-item {
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  padding: 16px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.team-item:hover {
  border-color: #16AEF3;
  box-shadow: 0 4px 12px rgba(22, 174, 243, 0.15);
}

.team-item.selected {
  border-color: #16AEF3;
  background: #e6f7ff;
  box-shadow: 0 4px 12px rgba(22, 174, 243, 0.25);
}

.team-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.team-name {
  font-size: 18px;
  font-weight: 600;
  color: #333;
}

.checkbox {
  width: 24px;
  height: 24px;
  border: 2px solid #d0d0d0;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
}

.checkbox.checked {
  background: #16AEF3;
  border-color: #16AEF3;
  color: white;
}

.team-preview {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 8px;
}

.character-slot {
  width: 72px;
  height: 72px;
  aspect-ratio: 1;
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fafafa;
  font-size: 12px;
  /* overflow: hidden; */
}

.character-preview-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.character-preview-wrapper .character-avatar-wrapper {
  width: 100%;
  height: 100%;
  transform: scale(1.0);
  transform-origin: center;
}

.empty-slot {
  color: #999;
}

/* Canvas内容样式 */
.no-selection {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #999;
  text-align: center;
}

.placeholder-icon {
  font-size: 48px;
  margin-bottom: 16px;
  opacity: 0.5;
}

.no-selection p {
  font-size: 16px;
  margin: 0;
}

.config-preview {
  margin-top: 20px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.config-info {
  font-size: 14px;
  font-weight: 600;
  margin-bottom: 8px;
  color: #666;
}

.config-detail {
  font-size: 12px;
  margin: 4px 0;
  color: #888;
}

.teams-canvas {
  position: relative;
  padding: 20px;
  user-select: none;
  width: fit-content;
}

.teams-canvas {
  border: 2px solid #007bff;
  border-radius: 12px;
}

/* PVP模式布局样式 */
.pvp-layout {
  display: flex;
  gap: 40px;
  width: fit-content;
}

.ally-teams-section,
.enemy-teams-section {
  width: fit-content;
  min-width: 0;
}

.section-header {
  margin: 0 0 20px 0;
  padding: 10px 15px;
  background: #f8f9fa;
  border-radius: 8px;
  font-size: 18px;
  font-weight: 600;
  color: #495057;
  text-align: center;
  border: 2px solid #dee2e6;
}

.enemy-header {
  background: #fff5f5;
  border-color: #fecaca;
  color: #dc2626;
}

.teams-grid {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.pvp-team {
  position: relative !important;
  left: auto !important;
  top: auto !important;
}

.enemy-team {
  border-color: #fecaca;
  background: #fff5f5;
}

.enemy-team .team-canvas-header h4 {
  color: #dc2626;
}

.team-canvas-item {
  width: fit-content;
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.2s ease;
}

.team-canvas-item:hover {
  border-color: #16AEF3;
  box-shadow: 0 6px 20px rgba(22, 174, 243, 0.2);
}

.team-canvas-header {
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid #e0e0e0;
}

.team-canvas-header h3 {
  margin: 0;
  color: #333;
  font-size: 18px;
  font-weight: 600;
}

.team-canvas-characters {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 12px;
  justify-content: center;
}

.character-canvas-slot {
  aspect-ratio: 1;
  border-radius: 8px;

  display: flex;

}

.character-canvas-wrapper {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
}

.character-canvas-wrapper .character-avatar-wrapper {
  width: 100%;
  height: 100%;
  transform: scale(1.0);
  transform-origin: center;
}

.empty-canvas-slot {
  width: 100%;
  height: 100%;
  background: #f5f5f5;
  border: 2px dashed #d0d0d0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  font-size: 12px;
  border-radius: 8px;
}



.preview-area {
  background: white;
  border-top: 1px solid #e0e0e0;
  padding: 20px;
  max-height: 40vh;
  overflow-y: auto;
}

.preview-area h2 {
  margin: 0 0 16px 0;
  color: #333;
  font-size: 20px;
  font-weight: 600;
  text-align: center;
}

.preview-container {
  max-width: 800px;
  margin: 0 auto;
}

.no-selection {
  text-align: center;
  color: #999;
  font-size: 16px;
  padding: 40px;
}

.selected-teams-preview {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.team-preview-item {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  padding: 16px;
  background: #fafafa;
}

.team-preview-item h3 {
  margin: 0 0 12px 0;
  color: #333;
  font-size: 16px;
  font-weight: 600;
}

.team-characters {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 8px;
}

.character-preview {
  aspect-ratio: 1;
  border: 1px solid #d0d0d0;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
  font-size: 12px;
}

.character-display {
  color: #16AEF3;
  font-weight: 500;
}

.empty-character {
  color: #999;
}

.action-bar {
  padding: 20px;
  background: #fafafa;
  border-top: 1px solid #e0e0e0;
  display: flex;
  justify-content: center;
  gap: 16px;
  margin-top: auto;
}

.btn {
  padding: 12px 24px;
  border: none;
  border-radius: 6px;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  min-width: 100px;
}

.btn-cancel {
  background: #3E4042;
  color: white;
  border: 1px solid #3E4042;
}

.btn-cancel:hover {
  background: #2a2c2e;
  border-color: #2a2c2e;
}

.btn-export {
  background: #16AEF3;
  color: white;
}

.btn-export:hover:not(:disabled) {
  background: #1890ff;
}

.btn-export:disabled {
  background: #d0d0d0;
  color: #999;
  cursor: not-allowed;
}

/* 手机端模式切换按钮 */
.mobile-mode-toggle {
  display: flex;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
  padding: 10px;
  gap: 0;
}

.mode-toggle-btn {
  flex: 1;
  padding: 12px 16px;
  background: white;
  border: 1px solid #dee2e6;
  color: #6c757d;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.mode-toggle-btn:first-child {
  border-radius: 6px 0 0 6px;
  border-right: none;
}

.mode-toggle-btn:last-child {
  border-radius: 0 6px 6px 0;
}

.mode-toggle-btn.active {
  background: #16AEF3;
  color: white;
  border-color: #16AEF3;
}

.mode-toggle-btn:hover:not(.active) {
  background: #f8f9fa;
  border-color: #16AEF3;
}

.mobile-hidden {
  display: none !important;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .export-image-fullscreen {
    flex-direction: column;
    height: 100vh;
  }

  .team-selection-panel {
    width: 100%;
    height: calc(100vh - 60px);
    max-height: none;
    border-right: none;
    border-bottom: none;
  }

  .canvas-preview-panel {
    width: 100%;
    height: calc(100vh - 60px);
  }

  /* 渲染配置移动端适配 */
  .render-config-section {
    padding: 10px;
    margin-bottom: 20px;
  }

  .section-title {
    font-size: 14px;
  }

  .config-subtitle {
    font-size: 12px;
  }

  .render-mode-selector {
    flex-direction: column;
    gap: 8px;
  }

  .render-mode-item {
    padding: 8px;
  }

  .mode-icon {
    font-size: 16px;
  }

  .mode-name {
    font-size: 11px;
  }



  .config-item {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  .config-item label {
    min-width: auto;
  }

  .range-input {
    width: 100%;
    margin: 0;
  }

  .team-type-tabs {
    flex-direction: column;
    gap: 6px;
  }

  .team-type-selector {
    gap: 8px;
    margin-bottom: 16px;
  }

  .team-type-item {
    min-width: 100px;
    padding: 8px 10px;
    font-size: 12px;
  }

  .team-type-name {
    font-size: 11px;
  }

  .panel-header,
  .canvas-header {
    padding: 15px;
  }

  .panel-header h2,
  .canvas-header h2 {
    font-size: 16px;
  }

  .team-list {
    padding: 15px;
    grid-template-columns: 1fr;
    max-width: 100%;
    overflow: visible;
  }

  .team-item {
    max-width: 100%;
    overflow: visible;
    overflow-x: auto;

  }



  .team-selection-panel {
    -webkit-overflow-scrolling: touch;
  }

  /* 自定义滚动条样式 */
  .team-selection-panel::-webkit-scrollbar {
    width: 6px;
  }

  .team-selection-panel::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 3px;
  }

  .team-selection-panel::-webkit-scrollbar-thumb {
    background: #c1c1c1;
    border-radius: 3px;
  }

  .team-selection-panel::-webkit-scrollbar-thumb:hover {
    background: #a8a8a8;
  }


  .character-preview-wrapper .character-avatar-wrapper {
    transform: scale(1.5);
  }

  .character-canvas-wrapper .character-avatar-wrapper {
    transform: scale(1.8);
  }




  .action-bar {
    padding: 15px;
  }

  .btn {
    padding: 10px 16px;
    font-size: 14px;
  }

  /* Canvas 控制按钮移动端优化 */
  .canvas-controls {
    gap: 8px;
  }

  .control-btn {
    padding: 6px;
    font-size: 14px;
    min-width: 32px;
    min-height: 32px;
    /* 增加触摸目标大小 */
    touch-action: manipulation;
  }

  .zoom-info {
    font-size: 12px;
    min-width: 40px;
  }

  /* Canvas 容器触摸优化 */
  .canvas-container {
    height: calc(50vh - 80px);
    /* 禁用双击缩放和其他触摸手势 */
    touch-action: none;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  .config-preview {
    padding: 10px;
  }

  .config-info {
    font-size: 12px;
  }

  .config-detail {
    font-size: 11px;
  }
}

/* 导出弹窗样式 */
.export-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10000;
  backdrop-filter: blur(4px);
}

.export-dialog {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
  width: 90%;
  max-width: 480px;
  max-height: 80vh;
  overflow: hidden;
  animation: dialogSlideIn 0.3s ease-out;
}

@keyframes dialogSlideIn {
  from {
    opacity: 0;
    transform: translateY(-20px) scale(0.95);
  }

  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.dialog-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
}

.dialog-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #111827;
}

.close-btn {
  background: none;
  border: none;
  padding: 4px;
  cursor: pointer;
  border-radius: 6px;
  color: #6b7280;
  transition: all 0.2s;
}

.close-btn:hover {
  background: #e5e7eb;
  color: #374151;
}

.dialog-content {
  padding: 24px;
  max-height: 60vh;
  overflow-y: auto;
}

.config-group {
  margin-bottom: 20px;
}

.config-group:last-child {
  margin-bottom: 0;
}

.config-label {
  display: block;
  font-size: 14px;
  font-weight: 500;
  color: #374151;
}

.config-input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  transition: border-color 0.2s;
}

.config-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.format-selector {
  display: flex;
  gap: 16px;
}

.format-option {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.format-radio {
  margin: 0;
}

.format-text {
  font-size: 14px;
  color: #374151;
}

.scale-selector {
  width: 100%;
}

.scale-selector,
.quality-selector {
  flex: 1;
  display: flex;
  align-items: center;
  gap: 12px;
}

.scale-range,
.quality-range {
  flex: 1;
  height: 6px;
  background: #e5e7eb;
  border-radius: 3px;
  outline: none;
}

.scale-range::-webkit-slider-thumb,
.quality-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  background: #3b82f6;
  border-radius: 50%;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.scale-value,
.quality-value {
  font-size: 14px;
  font-weight: 500;
  color: #374151;
  min-width: 40px;
  text-align: right;
}

.color-selector {
  display: flex;
  align-items: center;
  gap: 12px;
}

.color-input {
  width: 40px;
  height: 40px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  cursor: pointer;
  background: none;
  padding: 0;
}

.color-text {
  font-size: 14px;
  font-family: monospace;
  color: #374151;
}

.background-selector {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.background-type-tabs {
  display: flex;
  gap: 8px;
}

.background-type-item {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  background: white;
}

.background-type-item:hover {
  border-color: #3b82f6;
  background: #f8faff;
}

.background-type-item:has(.background-type-radio:checked) {
  border-color: #3b82f6;
  background: #eff6ff;
  color: #3b82f6;
}

.background-type-radio {
  margin: 0;
  accent-color: #3b82f6;
}

.background-type-text {
  font-size: 14px;
  font-weight: 500;
  user-select: none;
}

.transparent-preview {
  display: flex;
  align-items: center;
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  background: linear-gradient(45deg, #f0f0f0 25%, transparent 25%),
    linear-gradient(-45deg, #f0f0f0 25%, transparent 25%),
    linear-gradient(45deg, transparent 75%, #f0f0f0 75%),
    linear-gradient(-45deg, transparent 75%, #f0f0f0 75%);
  background-size: 8px 8px;
  background-position: 0 0, 0 4px, 4px -4px, -4px 0px;
}

.transparent-indicator {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #6b7280;
  font-size: 14px;
  font-weight: 500;
}

.transparent-indicator svg {
  opacity: 0.7;
}

/* 高级设置样式 */
.advanced-settings {
  margin-top: 20px;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
  background: white;
}

.advanced-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background: #f9fafb;
  border-bottom: 1px solid #e5e7eb;
  cursor: pointer;
  transition: all 0.2s;
  user-select: none;
}

.advanced-header:hover {
  background: #f3f4f6;
}

.advanced-header .config-subtitle {
  margin: 0;
  font-size: 14px;
  font-weight: 600;
  color: #374151;
}

.expand-icon {
  transition: transform 0.2s ease;
  color: #6b7280;
}

.expand-icon.expanded {
  transform: rotate(180deg);
}

.advanced-content {
  padding: 16px;
  background: white;
  border-top: 1px solid #e5e7eb;
}

.advanced-content .config-item {
  margin-bottom: 16px;
}

.advanced-content .config-item:last-child {
  margin-bottom: 0;
}

.select-input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 14px;
  background: white;
  color: #374151;
  transition: border-color 0.2s;
}

.select-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.range-input {
  width: calc(100% - 60px);
  margin-right: 12px;
  accent-color: #3b82f6;
}

.range-value {
  display: inline-block;
  min-width: 48px;
  font-size: 14px;
  font-weight: 500;
  color: #374151;
  text-align: right;
}

/* 优化复选框样式 */
.config-checkbox {
  appearance: none;
  width: 18px;
  height: 18px;
  border: 2px solid #d1d5db;
  border-radius: 4px;
  background: white;
  cursor: pointer;
  position: relative;
  transition: all 0.2s ease;
  margin-right: 8px;
  flex-shrink: 0;
}

.config-checkbox:hover {
  border-color: #3b82f6;
  background: #f8faff;
}

.config-checkbox:checked {
  background: #3b82f6;
  border-color: #3b82f6;
}

.config-checkbox:checked::after {
  content: '';
  position: absolute;
  left: 3px;
  top: 0px;
  width: 6px;
  height: 10px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.config-checkbox:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.checkbox-text {
  font-size: 14px;
  font-weight: 500;
  color: #374151;
  user-select: none;
  cursor: pointer;
}

.config-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  transition: color 0.2s;
}

.config-label:hover .checkbox-text {
  color: #3b82f6;
}

/* 水印位置控制样式 */
.watermark-position-controls {
  display: flex;
  gap: 12px;
  margin-top: 8px;
}

.position-input-group {
  display: flex;
  align-items: center;
  gap: 6px;
  flex: 1;
}

.position-input-group label {
  font-size: 12px;
  font-weight: 500;
  color: #6b7280;
  min-width: 16px;
}

.position-input {
  flex: 1;
  padding: 6px 8px;
  border: 1px solid #d1d5db;
  border-radius: 4px;
  font-size: 13px;
  background: white;
  color: #374151;
  transition: border-color 0.2s;
}

.position-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
}

.dialog-actions {
  display: flex;
  gap: 12px;
  padding: 20px 24px;
  border-top: 1px solid #e5e7eb;
  background: #f9fafb;
}

.dialog-actions .btn {
  flex: 1;
  padding: 10px 16px;
  border-radius: 6px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s;
}

.dialog-actions .btn-cancel {
  background: white;
  border: 1px solid #d1d5db;
  color: #374151;
}

.dialog-actions .btn-cancel:hover {
  background: #f9fafb;
  border-color: #9ca3af;
}

.dialog-actions .btn-export {
  background: #3b82f6;
  border: 1px solid #3b82f6;
  color: white;
}

.dialog-actions .btn-export:hover {
  background: #2563eb;
  border-color: #2563eb;
}

/* 进度弹窗样式 - 与导出弹窗风格一致 */
.progress-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10001;
  backdrop-filter: blur(4px);
}

.progress-dialog {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
  width: 90%;
  max-width: 420px;
  overflow: hidden;
  animation: dialogSlideIn 0.3s ease-out;
}

.progress-header {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px 24px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
}

.progress-header h3 {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: #111827;
}

.progress-content {
  padding: 32px 24px;
  text-align: center;
  background: white;
}

.progress-spinner {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.spinner {
  color: #3b82f6;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

.progress-message {
  margin: 0;
  font-size: 15px;
  color: #6b7280;
  font-weight: 500;
  line-height: 1.5;
}

/* 移动端弹窗适配 */
@media (max-width: 768px) {
  .export-dialog {
    width: 95%;
    max-width: none;
    margin: 20px;
  }

  .progress-dialog {
    width: 95%;
    max-width: none;
    margin: 20px;
  }

  .dialog-header,
  .dialog-actions {
    padding: 16px 20px;
  }

  .dialog-content {
    padding: 20px;
  }

  .progress-content {
    padding: 24px 20px;
  }

  .format-selector {
    flex-direction: column;
    gap: 12px;
  }

  .scale-selector,
  .quality-selector {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .scale-value,
  .quality-value {
    text-align: left;
  }
}

/* 水印样式 */
.watermark-layer {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  pointer-events: none;
  z-index: 10;
  overflow: hidden;
}

.watermark-draggable {
  position: absolute;
  pointer-events: auto;
  cursor: move;
  z-index: 15;
  transition: transform 0.1s ease;
}

.watermark-draggable:hover {
  transform: scale(1.05);
}

.watermark-draggable:active {
  transform: scale(0.95);
  cursor: grabbing;
}

.watermark-item.draggable-item {
  font-size: 14px;
  font-weight: 500;
  color: rgba(0, 0, 0, 0.4);
  background: rgba(255, 255, 255, 0.9);
  padding: 8px 12px;
  border-radius: 6px;
  backdrop-filter: blur(4px);
  border: 1px solid rgba(0, 0, 0, 0.1);
  user-select: none;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  white-space: nowrap;
  transition: all 0.2s ease;
}

.watermark-draggable:hover .draggable-item {
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  border-color: rgba(0, 0, 0, 0.2);
}

/* 深色模式水印适配 */
@media (prefers-color-scheme: dark) {
  .watermark-item.draggable-item {
    color: rgba(255, 255, 255, 0.7);
    background: rgba(0, 0, 0, 0.8);
    border-color: rgba(255, 255, 255, 0.1);
  }

  .watermark-draggable:hover .draggable-item {
    background: rgba(0, 0, 0, 0.9);
    border-color: rgba(255, 255, 255, 0.2);
  }
}

/* 样式选择器样式 */
.style-selector {
  display: flex;
  gap: 16px;
  margin-top: 8px;
}

.style-option {
  flex: 1;
}

.style-label {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
  background: white;
  font-size: 14px;
  font-weight: 500;
  color: #374151;
}

.style-label:hover {
  border-color: #3b82f6;
  background: #f0f9ff;
}

.style-radio {
  width: 16px;
  height: 16px;
  margin: 0;
  accent-color: #3b82f6;
}

.style-radio:checked+.style-text {
  color: #3b82f6;
  font-weight: 600;
}

.style-label:has(.style-radio:checked) {
  border-color: #3b82f6;
  background: #eff6ff;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.1);
}

.style-text {
  transition: color 0.2s ease;
}

.character-card {
  width: 180px;
  height: 305px;
  overflow: hidden;
  position: relative;
  filter: drop-shadow(0 6px 12px rgba(0, 0, 0, 0.3));
  /* 使用scale控制缩放以适配更多列 */
  --scale-factor: 0.9;
  transform: scale(var(--scale-factor));
  transform-origin: center;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.character-mask {
  width: 100%;

  position: relative;
  bottom: -19px;
}

/* 移动端样式选择器适配 */
@media (max-width: 768px) {
  .style-selector {
    flex-direction: column;
    gap: 12px;
  }

  .style-label {
    padding: 10px 14px;
    font-size: 13px;
  }
}

/* 样式编辑器弹窗样式 */
.style-editor-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  backdrop-filter: blur(4px);
}

.style-editor-dialog {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
  width: 90%;
  max-width: 800px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.style-editor-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid #e5e7eb;
  background: #f9fafb;
}

.style-editor-header h3 {
  margin: 0;
  color: #111827;
  font-size: 18px;
  font-weight: 600;
}

.style-editor-content {
  flex: 1;
  overflow: hidden;
}

.style-editor-actions {
  padding: 16px 24px;
  border-top: 1px solid #e5e7eb;
  background: #f9fafb;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.btn-style-editor {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: #f3f4f6;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  color: #374151;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-style-editor:hover {
  background: #e5e7eb;
  border-color: #9ca3af;
  color: #111827;
}

.btn-style-editor:active {
  transform: translateY(1px);
}

.btn-style-editor svg {
  width: 14px;
  height: 14px;
}

/* 移动端样式编辑器适配 */
@media (max-width: 768px) {
  .style-editor-dialog {
    width: 100%;
    max-width: none;
    height: 100%;
    border-radius: 0;
  }

  .style-editor-header {
    padding: 5px 20px;
  }

  .style-editor-header h3 {
    font-size: 16px;
  }

  .style-editor-actions {
    padding: 12px 20px;
  }

  .btn-style-editor {
    padding: 6px 12px;
    font-size: 12px;
  }
}

/* 详细信息配置样式 */

.detail-options {
  margin-top: 15px;
}

.detail-category {
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 6px;
  border: 1px solid #dee2e6;
}

.detail-category-title {
  margin: 0 0 12px 0;
  font-size: 14px;
  font-weight: 600;
  color: #495057;
  border-bottom: 1px solid #e9ecef;
  padding-bottom: 8px;
}

.detail-items {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 10px;
}

.detail-item-label {
  display: flex;
  align-items: center;
  cursor: pointer;
  font-size: 13px;
  color: #495057;
  padding: 8px;
  border-radius: 4px;
  transition: background-color 0.2s ease;
}

.detail-item-label:hover {
  background-color: #f8f9fa;
}

.detail-checkbox {
  margin-right: 6px;
  accent-color: #007bff;
}

.detail-text {
  font-weight: 500;
}

.custom-info-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.custom-info-item {
  padding: 15px;
  background: #f8f9fa;
  border-radius: 6px;
  border: 1px solid #e9ecef;
}

.custom-info-controls {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-bottom: 10px;
}

.custom-input {
  padding: 6px 10px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 13px;
  transition: border-color 0.2s ease;
}

.custom-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.custom-label {
  flex: 0 0 100px;
}

.custom-value {
  flex: 1;
}

.custom-color {
  width: 40px;
  height: 32px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  cursor: pointer;
  background: none;
  padding: 0;
}

.remove-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.remove-btn:hover {
  background: #c82333;
}

.custom-info-image {
  display: flex;
  align-items: center;
  gap: 10px;
}

.image-upload-label {
  display: inline-block;
  padding: 6px 12px;
  background: #007bff;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: background-color 0.2s ease;
}

.image-upload-label:hover {
  background: #0056b3;
}

.image-upload-input {
  display: none;
}

.image-upload-text {
  font-weight: 500;
}

.image-preview {
  position: relative;
  display: inline-block;
}

.custom-image {
  width: 40px;
  height: 40px;
  object-fit: cover;
  border-radius: 4px;
  border: 1px solid #dee2e6;
}

.remove-image-btn {
  position: absolute;
  top: -6px;
  right: -6px;
  width: 20px;
  height: 20px;
  background: #dc3545;
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
  transition: background-color 0.2s ease;
}

.remove-image-btn:hover {
  background: #c82333;
}

.add-custom-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  width: 100%;
  padding: 12px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background-color 0.2s ease;
}

.add-custom-btn:hover {
  background: #218838;
}

.detail-style-config {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.select-input {
  padding: 6px 10px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  font-size: 13px;
  background: white;
  cursor: pointer;
}

.select-input:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.25);
}

.scale-selector {
  display: flex;
  align-items: center;
  gap: 10px;
}

.scale-range {
  flex: 1;
  height: 6px;
  background: #e9ecef;
  border-radius: 3px;
  outline: none;
  cursor: pointer;
}

.scale-range::-webkit-slider-thumb {
  appearance: none;
  width: 16px;
  height: 16px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
}

.scale-range::-moz-range-thumb {
  width: 16px;
  height: 16px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  border: none;
}

.scale-value {
  font-size: 12px;
  font-weight: 500;
  color: #6c757d;
  min-width: 40px;
  text-align: right;
}

.color-input {
  width: 60px;
  height: 32px;
  border: 1px solid #ced4da;
  border-radius: 4px;
  cursor: pointer;
  background: none;
  padding: 0;
}

/* 详细信息显示样式 */
.team-detail-info {
  margin-top: 16px;
  padding: 12px;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.character-detail {
  background: rgba(255, 255, 255, 0.9);
  border-radius: 6px;
  padding: 10px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.character-detail-header {
  margin-bottom: 8px;
  padding-bottom: 6px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.character-name {
  font-weight: bold;
  font-size: 14px;
  color: #333;
}

.character-detail-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.detail-section {
  margin-bottom: 6px;
}

.detail-row {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  font-size: 12px;
  line-height: 1.4;
}

.detail-label {
  font-weight: 500;
  color: #666;
  min-width: 50px;
  flex-shrink: 0;
}

.detail-value {
  color: #333;
  flex: 1;
}

/* 魔方显示样式 */
.cube-display {
  display: flex;
  align-items: center;
  gap: 6px;
}

.cube-image-small {
  width: 48px;
  object-fit: cover;
  border-radius: 3px;
}

.cube-level {
  font-size: 11px;
  color: #333;
}

/* 收藏品显示样式 */
.favorite-display {
  display: flex;
  align-items: center;
  gap: 6px;
}

.favorite-image-small {
  width: 32px;
  object-fit: cover;
  border-radius: 3px;
}

.favorite-level {
  font-size: 11px;
  color: #333;
}

/* 装备网格显示样式 */
.equipment-grid-small {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(60px, 1fr));
  gap: 8px;
  max-width: 300px;
}

.equipment-item-small {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.equipment-card-small {
  position: relative;
  width: 48px;
  height: 48px;
  border-radius: 4px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.2s ease;
}

.equipment-card-small.t10 {
  box-shadow: 0 0 8px rgba(255, 215, 0, 0.6);
}

.equipment-background-small {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 1;
}

.equipment-icon-small {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 44px;
  height: 44px;
  object-fit: cover;
  z-index: 2;
}

.equipment-tier-overlay-small {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 3;
  pointer-events: none;
}

.equipment-tier-overlay-small.t9 {
  background: linear-gradient(45deg, rgba(138, 43, 226, 0.2) 0%, rgba(138, 43, 226, 0.1) 100%);
}

.equipment-tier-overlay-small.t10 {
  background: linear-gradient(45deg, rgba(255, 215, 0, 0.3) 0%, rgba(255, 215, 0, 0.1) 100%);
}

.equipment-level-small {
  position: absolute;
  bottom: 2px;
  right: 2px;
  color: white;
  font-size: 8px;
  padding: 1px 3px;
  border-radius: 2px;
  z-index: 4;
}

.equipment-name-small {
  font-size: 10px;
  color: #666;
  text-align: center;
  line-height: 1.2;
}

/* 词条网格显示样式 */
.entries-grid {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.part-entries {
  background: rgba(248, 249, 250, 0.8);
  border-radius: 4px;
  padding: 6px;
  border: 1px solid rgba(0, 0, 0, 0.1);
}

.part-header {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 4px;
  padding-bottom: 4px;
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.part-icon {
  width: 20px;
  height: 20px;
  object-fit: cover;
  border-radius: 2px;
}

.part-name {
  font-weight: 500;
  color: #666;
  font-size: 11px;
}

.entries-list {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.entry-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: rgba(40, 167, 69, 0.1);
  color: #28a745;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 10px;
}

.entry-type {
  font-weight: 500;
}

.entry-value {
  font-weight: bold;
}

/* 自定义信息样式 */
.custom-content {
  display: flex;
  align-items: center;
  gap: 6px;
}

.custom-info-image {
  width: 24px;
  height: 24px;
  object-fit: cover;
  border-radius: 3px;
}

.equipment-info,
.entries-info {
  width: 100%;
}

.equipment-list {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.equipment-item {
  color: #007bff;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 11px;
  white-space: nowrap;
}

.custom-info {
  align-items: center;
}

/* 装备显示样式 */
.equipment-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
  flex: 1;
}

.equipment-item {
  display: flex;
  transform: scale(0.8);
  flex-direction: column;
  align-items: center;
}

.equipment-label {
  font-weight: bold;
  color: #495057;
  margin-bottom: 8px;
  text-align: center;
  font-size: 14px;
}

.equipment-card {
  position: relative;
  width: 90px;
  height: 90px;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.2);
}

.equipment-card.t10::before {
  content: '';
  z-index: 99;
  width: 100%;
  height: 100%;
  position: absolute;
  inset: 0;
  box-shadow: inset 0 0 10px #fa498160;
}

.equipment-card.t10::after {
  content: '';
  position: absolute;
  top: -100%;
  z-index: 999;
  left: -100%;
  width: 200%;
  height: 200%;
  background: linear-gradient(135deg,
      rgba(255, 255, 255, 0) 40%,
      rgba(255, 255, 255, 0.9) 50%,
      rgba(255, 255, 255, 0) 60%);
  animation: sweep-light 4s linear infinite;
  z-index: 100;
  pointer-events: none;
  filter: blur(10px);
  opacity: 0.8;
}

@keyframes sweep-light {
  0% {
    top: -120%;
    left: -120%;
  }

  100% {
    top: 110%;
    left: 110%;
  }
}

.equipment-background {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 1;
}

.equipment-icon {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  z-index: 2;
}

.equipment-tier-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 15px;
  opacity: 0.8;
  z-index: 3;
}

.equipment-tier-overlay.t9 {
  background: linear-gradient(to top, #F4973A, transparent);
}

.equipment-tier-overlay.t10 {
  box-shadow: 0px;
  background: linear-gradient(to top, #FA4981, transparent);
}

.equipment-level {
  position: absolute;
  display: flex;
  flex-direction: column;
  left: 4px;
  bottom: 0;
  z-index: 10;
  color: white;
  font-size: 18px;
  font-weight: normal !important;
  text-shadow:
    -1px -1px 1px rgba(0, 0, 0, 0.5),
    1px -1px 1px rgba(0, 0, 0, 0.5),
    -1px 1px 1px rgba(0, 0, 0, 0.5),
    1px 1px 1px rgba(0, 0, 0, 0.5),
    0px -1px 1px rgba(0, 0, 0, 0.5),
    0px 1px 1px rgba(0, 0, 0, 0.5),
    -1px 0px 1px rgba(0, 0, 0, 0.5),
    1px 0px 1px rgba(0, 0, 0, 0.5);
}

.equipment-ct {
  user-select: none;
  width: 160px;;
  padding: 0 4px;
  padding-right: 7px;
  background-color: #E4E4E9;
  border-radius: 5px;
  margin: 3px;
  color: black;
  font-weight: normal !important;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border: 1px solid transparent;
}

/* 移动端详细信息配置适配 */
@media (max-width: 768px) {
  .detail-items {
    grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
    gap: 8px;
  }

  .custom-info-controls {
    flex-direction: column;
    align-items: stretch;
    gap: 8px;
  }

  .custom-label,
  .custom-value {
    flex: none;
  }

  .detail-style-config {
    gap: 10px;
  }

  .config-item {
    flex-direction: column;
    align-items: stretch;
    gap: 6px;
  }

  .scale-selector {
    gap: 8px;
  }

  .team-detail-info {
    margin-top: 12px;
    padding: 8px;
    gap: 8px;
  }

  .character-detail {
    padding: 8px;
  }

  .character-name {
    font-size: 13px;
  }

  .detail-row {
    font-size: 11px;
    gap: 6px;
  }

  .detail-label {
    min-width: 40px;
  }

  .equipment-item,
  .entry-item {
    font-size: 10px;
    padding: 1px 4px;
  }

  .part-name {
    font-size: 10px;
  }

  .custom-info .custom-info-image {
    width: 20px;
    height: 20px;
  }
}
</style>