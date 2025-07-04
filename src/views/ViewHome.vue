<template>
  <div class="w-full h-screen flex flex-col items-center justify-center relative overflow-hidden">
    <!-- Restore original background -->
    <div class="absolute inset-0 w-full h-full z-0">
      <img src="/assets/ui-assets/recipes-large-bg.png" alt="Background" class="w-full h-full object-cover" draggable="false" />
    </div>
    <div class="absolute inset-0 w-full h-full z-10 overlay-blur"></div>
    
    <div class="absolute top-2 right-2 z-40 flex gap-4">
      <button 
        @click="showBackupModal = true"
        class="action-btn-compact bg-[#058696] hover:bg-[#047a8a] text-white"
      >
        <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <ellipse cx="12" cy="6" rx="8" ry="3" stroke-width="2" />
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M4 6v6c0 1.657 3.582 3 8 3s8-1.343 8-3V6" />
          <path stroke-width="2" stroke-linecap="round" stroke-linejoin="round" d="M4 12v6c0 1.657 3.582 3 8 3s8-1.343 8-3v-6" />
        </svg>
        Backup
      </button>
      <button 
        @click="goToAddRecipe" 
        class="action-btn-compact bg-[#F1362F] hover:bg-[#d42e28] text-white"
      >
        <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"/>
        </svg>
        Add
      </button>
    </div>
    
    <MenuBar class="absolute top-0 left-0 w-full z-30" :currentView="'recipes'" :showRupees="false" 
      @switch-adventure-log="goToAdventureLog"
    />
    
    <div class="relative z-20 flex flex-col items-center w-full h-full pt-24 pb-8 px-6">
      <!-- Main Content Area with Recipe Grid and Preview -->
      <div class="flex-1 w-full max-w-7xl flex gap-4">
        <!-- Recipe Grid and Search Container -->
        <div class="flex-1 overflow-y-auto">
          <!-- Search Bar - Aligned with Recipe Grid -->
          <div class="mb-4 ml-20">
            <div class="relative max-w-md search-container ml-2">
              <input 
                type="text" 
                v-model="searchQuery"
                placeholder="Search recipes..." 
                class="search-input-bar w-full h-[40px] bg-[#D9D9D9]/95 text-lg text-[#222] font-semibold px-4 pr-20 rounded-lg shadow-lg focus:outline-none focus:ring-2 focus:ring-[#333] transition-all duration-300" 
                style="border:none;" 
              />
              <svg class="absolute right-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-[#333] pointer-events-none" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                <circle cx="11" cy="11" r="8"/>
                <line x1="21" y1="21" x2="16.65" y2="16.65"/>
              </svg>
              
              <!-- Filter Button -->
              <button 
                @click="toggleFilterDropdown"
                class="absolute right-10 top-1/2 transform -translate-y-1/2 w-6 h-6 text-[#333] hover:text-[#047a8a] transition-colors duration-200"
                :class="{ 'text-[#047a8a]': selectedFilters.length > 0 }"
              >
                <svg fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/>
                  <path stroke-linecap="round" stroke-linejoin="round" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                </svg>
                <!-- Filter count badge -->
                <span v-if="selectedFilters.length > 0" 
                      class="absolute -top-1 -right-1 bg-[#047a8a] text-white text-xs rounded-full w-4 h-4 flex items-center justify-center">
                  {{ selectedFilters.length }}
                </span>
              </button>
              
              <!-- Filter Dropdown -->
              <div v-if="showFilterDropdown" class="absolute top-full right-0 mt-2 bg-[#E8E8E8]/95 backdrop-blur-sm border border-white/20 rounded-lg shadow-lg z-50 w-1/2">
                <div class="p-3 space-y-2">
                  <div class="flex justify-between items-center mb-2">
                    <div class="text-sm font-semibold text-[#222]">Filter by:</div>
                    <button 
                      v-if="selectedFilters.length > 0"
                      @click="selectedFilters = []"
                      class="text-xs text-[#047a8a] hover:text-[#058696] transition-colors duration-200"
                    >
                      Clear all
                    </button>
                  </div>
                  <label 
                    v-for="filter in filters" 
                    :key="filter.name"
                    class="flex items-center space-x-2 cursor-pointer hover:bg-white/50 rounded px-2 py-1 transition-colors duration-200"
                  >
                    <input 
                      type="checkbox" 
                      :checked="selectedFilters.includes(filter.name)"
                      @change="toggleFilter(filter.name)"
                      class="w-4 h-4 text-[#058696] rounded focus:ring-[#058696]"
                    />
                    <span class="text-sm text-[#222]">{{ filter.name }}</span>
                  </label>
                </div>
              </div>
            </div>
          </div>

          <!-- Recipe Grid -->
          <div class="recipe-grid-scroll ml-20 max-w-md">
            <div 
              v-for="(recipe, index) in filteredRecipes" 
              :key="recipe.id"
              class="recipe-square bg-white/10 backdrop-blur-sm border border-white/20 rounded-md p-2 hover:bg-white/20 transition-all duration-300 cursor-pointer group min-w-[80px]"
              @click="selectRecipe(recipe)"
              @mouseenter="showPreview(recipe, index)"
              @mouseleave="hidePreview"
            >
              <!-- Recipe Image - Tiny Square -->
              <div class="relative w-full aspect-square mb-2 rounded-sm overflow-hidden">
                <img 
                  :src="getRecipeImageUrl(recipe)" 
                  :alt="recipe.name"
                  class="w-full h-full object-cover group-hover:scale-105 transition-transform duration-300"
                />
                <div class="absolute inset-0 bg-gradient-to-t from-black/30 to-transparent"></div>
              </div>
              
              <!-- Recipe Info -->
              <div class="space-y-1">
                <h3 class="text-[10px] font-bold text-white transition-colors duration-300 line-clamp-1 leading-tight">
                  {{ recipe.name }}
                </h3>
                
                <!-- Hearts Rating -->
                <div class="flex justify-start">
                  <div class="flex space-x-0.5">
                    <img
                      v-for="n in 5"
                      :key="n"
                      :src="getHeartSrc(n, recipe.hearts)"
                      alt="heart"
                      class="w-2 h-2"
                    />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Recipe Preview Panel -->
        <div class="w-80 flex-shrink-0">
          <div 
            v-if="previewRecipe"
            class="sticky top-24 transition-all duration-300 ease-out"
            :class="previewVisible ? 'opacity-100 translate-x-0' : 'opacity-0 translate-x-4'"
            @mouseenter="isHoveringPreview = true"  
            @mouseleave="isHoveringPreview = false"
          >
            <RecipeCard 
              :recipe="previewRecipe" 
              :isActive="true"
              :index="previewIndex"
              class="preview-recipe-card"
            />
          </div>
        </div>
      </div>
    </div>

    <!-- Full Recipe Card Popup -->
    <FullRecipeCard 
      v-if="showFullRecipe && !showEditForm" 
      :recipe="selectedRecipe" 
      :closing="fullRecipeClosing" 
      @close="handleFullRecipeClose"
      @delete="handleRecipeDelete"
      @edit="handleRecipeEdit"
    />

    <EditRecipeForm
      v-if="showEditForm"
      :recipe="editingRecipe"
      @back-to-home="handleEditFormClose"
      @updated="() => { handleEditFormClose(); refreshRecipes(); }"
    />

    <BackupModal
      v-if="showBackupModal"
      @close="showBackupModal = false"
      @recipes-imported="handleRecipesImported"
    />

    <QuestCompletion
      :visible="showQuestCompletion"
      :questId="completedQuestId"
      @close="showQuestCompletion = false"
    />
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue';
import MenuBar from '../components/MenuBar.vue';
import { Recipe } from '../models/Recipe.js';
import FullRecipeCard from '../components/FullRecipeCard.vue';
import RecipeCard from '../components/RecipeCard.vue';
import { loadRecipes, deleteRecipe } from '../scripts/recipesStorage.js';
import EditRecipeForm from '../components/EditRecipeForm.vue';
import { useRouter } from 'vue-router';
import BackupModal from '../components/BackupModal.vue';
import QuestCompletion from '../components/QuestCompletion.vue';
import { updateQuestState, getQuestById } from '../data/quests.js';

export default {
  name: 'ViewHome',
  components: { MenuBar, FullRecipeCard, RecipeCard, BackupModal, QuestCompletion },
  setup() {
    const searchQuery = ref('');
    const selectedFilters = ref([]);
    const showFullRecipe = ref(false);
    const selectedRecipe = ref(null);
    const fullRecipeClosing = ref(false);
    const showFilterDropdown = ref(false);
    const previewRecipe = ref(null);
    const previewIndex = ref(0);
    const previewVisible = ref(false);
    const isHoveringPreview = ref(false);
    const router = useRouter();
    const showBackupModal = ref(false);
    const showEditForm = ref(false);
    const editingRecipe = ref(null);
    const showQuestCompletion = ref(false);
    const completedQuestId = ref('quest-1');
    
    // Check if quest completion has been shown before
    const hasShownQuestCompletion = ref(false);
    const hasShownQuest2Completion = ref(false);
    if (typeof window !== 'undefined') {
      hasShownQuestCompletion.value = localStorage.getItem('quest-1-completion-shown') === 'true';
      hasShownQuest2Completion.value = localStorage.getItem('quest-2-completion-shown') === 'true';
    }
    
    const filters = [
      {
        name: 'Breakfast',
        icon: 'M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z'
      },
      {
        name: 'Entrée',
        icon: 'M8.1 13.34l2.83-2.83L3.91 3.5c-1.56 1.56-1.56 4.09 0 5.66l4.19 4.18zm6.78-1.81c1.53.71 3.68.21 5.27-1.38 1.91-1.91 2.28-4.65.81-6.12-1.46-1.46-4.2-1.1-6.12.81-1.59 1.59-2.09 3.74-1.38 5.27L3.7 19.87l1.41 1.41L12 14.41l6.88 6.88 1.41-1.41L13.41 13l1.47-1.47z'
      },
      {
        name: 'Dessert',
        icon: 'M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z'
      }
    ];

    // Remove hardcoded recipes, use localStorage
    const recipes = ref([]);
    async function refreshRecipes() {
      const previousCount = recipes.value.length;
      recipes.value = await loadRecipes();
      
      // Check if we should prevent quest completion popup (after import)
      const preventQuestPopup = typeof window !== 'undefined' && localStorage.getItem('prevent-quest-popup') === 'true';
      
      // Check if recipe count just became 1 and complete quest-1
      if (previousCount === 0 && recipes.value.length === 1) {
        updateQuestState('quest-1', 'complete');
        // Activate quest-2 when quest-1 is completed
        updateQuestState('quest-2', 'active');
        // Show quest completion overlay only if this is the first time completing the quest
        // AND we're not preventing popup after import
        if (!hasShownQuestCompletion.value && !preventQuestPopup) {
          completedQuestId.value = 'quest-1';
          showQuestCompletion.value = true;
          hasShownQuestCompletion.value = true;
          // Mark as shown in localStorage
          if (typeof window !== 'undefined') {
            localStorage.setItem('quest-1-completion-shown', 'true');
          }
        }
      }
      // Check if quest-1 should be completed (has at least 1 recipe and is currently active)
      else if (recipes.value.length >= 1 && getQuestById('quest-1')?.state === 'active') {
        updateQuestState('quest-1', 'complete');
        // Activate quest-2 when quest-1 is completed
        updateQuestState('quest-2', 'active');
        // Show quest completion overlay only if this is the first time completing the quest
        // AND we're not preventing popup after import
        if (!hasShownQuestCompletion.value && !preventQuestPopup) {
          completedQuestId.value = 'quest-1';
          showQuestCompletion.value = true;
          hasShownQuestCompletion.value = true;
          // Mark as shown in localStorage
          if (typeof window !== 'undefined') {
            localStorage.setItem('quest-1-completion-shown', 'true');
          }
        }
      }
      // Check if recipe count went back to 0 and reset quest-1 to active
      else if (recipes.value.length === 0) {
        updateQuestState('quest-1', 'active');
        // Hide quest-2 when quest-1 becomes active again
        updateQuestState('quest-2', 'hidden');
        // Reset the quest completion shown flag when quest becomes active again
        hasShownQuestCompletion.value = false;
        if (typeof window !== 'undefined') {
          localStorage.removeItem('quest-1-completion-shown');
        }
      }
      
      // Check if quest-2 should be completed (has at least one recipe of each category)
      const categories = ['Breakfast', 'Entrée', 'Dessert'];
      const hasAllCategories = categories.every(category => 
        recipes.value.some(recipe => recipe.category === category)
      );
      
      // Get current quest states
      const currentQuest2State = getQuestById('quest-2')?.state;
      const currentQuest1State = getQuestById('quest-1')?.state;
      
      // Debug logging
      console.log('Quest-2 Debug:', {
        hasAllCategories,
        currentQuest2State,
        quest1State: currentQuest1State,
        recipes: recipes.value.map(r => ({ name: r.name, category: r.category })),
        hasShownQuest2Completion: hasShownQuest2Completion.value
      });
      
      // If quest-1 is complete but quest-2 is hidden, activate quest-2
      if (currentQuest1State === 'complete' && currentQuest2State === 'hidden') {
        updateQuestState('quest-2', 'active');
        console.log('Quest-2 activated!');
      }
      
      // Get updated quest-2 state after potential activation
      const updatedQuest2State = getQuestById('quest-2')?.state;
      
      console.log('Updated Quest-2 state:', updatedQuest2State);
      
      if (hasAllCategories && updatedQuest2State === 'active') {
        updateQuestState('quest-2', 'complete');
        // Show quest-2 completion overlay if this is the first time completing it
        // AND we're not preventing popup after import
        if (!hasShownQuest2Completion.value && !preventQuestPopup) {
          completedQuestId.value = 'quest-2';
          showQuestCompletion.value = true;
          hasShownQuest2Completion.value = true;
          // Mark as shown in localStorage
          if (typeof window !== 'undefined') {
            localStorage.setItem('quest-2-completion-shown', 'true');
          }
        }
      }
      
      // Clear the prevent popup flag after processing quests
      if (preventQuestPopup && typeof window !== 'undefined') {
        localStorage.removeItem('prevent-quest-popup');
      }
    }
    refreshRecipes();

    // Listen for recipe-added event
    if (typeof window !== 'undefined') {
      window.addEventListener('recipe-added', refreshRecipes);
    }

    const filteredRecipes = computed(() => {
      return recipes.value.filter(recipe => {
        const matchesSearch = searchQuery.value === '' || 
          recipe.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
          recipe.description.toLowerCase().includes(searchQuery.value.toLowerCase());
        
        const matchesFilter = selectedFilters.value.length === 0 || 
          selectedFilters.value.some(filter => {
            // Check if recipe has a category and it matches the selected filter
            return recipe.category && recipe.category === filter;
          });
        
        return matchesSearch && matchesFilter;
      });
    });

    function toggleFilter(filter) {
      const idx = selectedFilters.value.indexOf(filter);
      if (idx === -1) {
        selectedFilters.value.push(filter);
      } else {
        selectedFilters.value.splice(idx, 1);
      }
    }

    function getAssetPath(path) {
      // Check if we're on GitHub Pages by looking at the hostname
      if (window.location.hostname.includes('github.io')) {
        // For GitHub Pages, we need to add the repository name prefix
        return `/sheikah-plate${path}`;
      }
      // Local development and preview - use relative paths
      return path;
    }

    function getHeartSrc(n, hearts) {
      // Default to GitHub Pages paths, but allow override for local development
      const isLocal = window.location.hostname === 'localhost' || window.location.hostname === '127.0.0.1';
      const basePath = isLocal ? '' : '/sheikah-plate';
      
      if (hearts >= n) {
        return `${basePath}/assets/ui-assets/full-heart.png`;
      }
      if (hearts >= n - 0.5) {
        return `${basePath}/assets/ui-assets/half-heart.png`;
      }
      return `${basePath}/assets/ui-assets/empty-heart.png`;
    }

    function getRecipeImageUrl(recipe) {
      if (!recipe.picture) return '';
      
      // Handle string URLs (base64 data URLs or regular URLs)
      if (typeof recipe.picture === 'string') {
        return recipe.picture;
      }
      
      // Handle Blob objects
      if (recipe.picture instanceof Blob || recipe.picture instanceof File) {
        try {
          return URL.createObjectURL(recipe.picture);
        } catch (error) {
          console.warn('Error creating object URL for recipe:', recipe.name, error);
          return '';
        }
      }
      
      // Handle other object types that might have been serialized
      if (typeof recipe.picture === 'object' && recipe.picture !== null) {
        console.warn('Unexpected picture object type for recipe:', recipe.name, recipe.picture);
        return '';
      }
      
      // If it's not a valid type, return empty string
      console.warn('Invalid picture type for recipe:', recipe.name, typeof recipe.picture);
      return '';
    }

    function selectRecipe(recipe) {
      selectedRecipe.value = recipe;
      showFullRecipe.value = true;
    }

    function showPreview(recipe, index) {
      previewRecipe.value = recipe;
      previewIndex.value = index;
      previewVisible.value = true;
    }

    function hidePreview() {
      setTimeout(() => {
        const hoveredElement = document.querySelector('.recipe-square:hover');
        if (!hoveredElement && !isHoveringPreview.value) {
          previewVisible.value = false;
          setTimeout(() => {
            previewRecipe.value = null;
          }, 300);
        }
      }, 50);
    }

    function handleFullRecipeClose() {
      fullRecipeClosing.value = true;
      setTimeout(() => {
        showFullRecipe.value = false;
        selectedRecipe.value = null;
        fullRecipeClosing.value = false;
      }, 300);
    }

    function handleRecipeDelete(recipe) {
      deleteRecipe(recipe.id);
      refreshRecipes();
      // Close the full recipe modal
      handleFullRecipeClose();
    }

    function handleKeyPress(event) {
      // If the event target is an input, textarea, or contenteditable element, 
      // don't handle navigation keys to allow normal typing
      if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA' || event.target.contentEditable === 'true') {
        return;
      }
      
      // Only handle navigation if we're actually on the home route
      if (router.currentRoute.value.name !== 'Home') {
        return;
      }
      
      if (showFullRecipe.value) {
        handleFullRecipeClose();
        return;
      }
      if (
        event.key === 'l' || event.key === 'L' ||
        event.key === 'ArrowLeft'
      ) {
        goToAdventureLog();
      }
    }

    function handleClickOutside(event) {
      const searchContainer = event.target.closest('.search-container');
      if (!searchContainer && showFilterDropdown.value) {
        showFilterDropdown.value = false;
      }
    }

    function toggleFilterDropdown() {
      showFilterDropdown.value = !showFilterDropdown.value;
    }

    function handleRecipeEdit(recipe) {
      router.push({ name: 'EditRecipe', params: { id: recipe.id } });
    }

    function handleEditFormClose() {
      showEditForm.value = false;
      editingRecipe.value = null;
    }

    function goToRecipes() {
      router.push({ name: 'Recipes' });
    }
    function goToAddRecipe() {
      router.push({ name: 'AddRecipe' });
    }
    function goToAdventureLog() {
      router.push({ name: 'AdventureLog' });
    }

    function handleRecipesImported() {
      // Set a flag to prevent quest completion popup after page refresh
      if (typeof window !== 'undefined') {
        localStorage.setItem('prevent-quest-popup', 'true');
        window.location.reload();
      }
    }

    onMounted(() => {
      document.addEventListener('keydown', handleKeyPress);
      document.addEventListener('click', handleClickOutside);
    });

    onUnmounted(() => {
      document.removeEventListener('keydown', handleKeyPress);
      document.removeEventListener('click', handleClickOutside);
      if (typeof window !== 'undefined') {
        window.removeEventListener('recipe-added', refreshRecipes);
      }
    });

    return {
      searchQuery,
      filters,
      selectedFilters,
      recipes,
      filteredRecipes,
      toggleFilter,
      getHeartSrc,
      selectRecipe,
      showFullRecipe,
      selectedRecipe,
      fullRecipeClosing,
      handleFullRecipeClose,
      handleRecipeDelete,
      showFilterDropdown,
      toggleFilterDropdown,
      previewRecipe,
      previewIndex,
      previewVisible,
      showPreview,
      hidePreview,
      getRecipeImageUrl,
      isHoveringPreview,
      handleRecipeEdit,
      goToRecipes,
      goToAddRecipe,
      goToAdventureLog,
      showBackupModal,
      handleRecipesImported,
      showEditForm,
      editingRecipe,
      showQuestCompletion,
      completedQuestId
    };
  }
}
</script>

<style scoped>
.overlay-blur {
  background: rgba(0, 0, 0, 0.30);
  backdrop-filter: blur(5px);
}

.search-input-bar::-webkit-input-placeholder {
  color: #666;
  font-weight: 500;
}
.search-input-bar::-moz-placeholder {
  color: #666;
  font-weight: 500;
}
.search-input-bar:-ms-input-placeholder {
  color: #666;
  font-weight: 500;
}
.search-input-bar::placeholder {
  color: #666;
  font-weight: 500;
}

.action-btn-compact {
  display: flex;
  align-items: center;
  font-size: 0.875rem;
  font-weight: 600;
  border: 1px solid transparent;
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  transition: all 0.3s;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  white-space: nowrap;
}

.action-btn-compact:hover {
  transform: translateY(-1px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

.line-clamp-1 {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.recipe-square {
  backdrop-filter: blur(15px);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.08);
  margin-top: 6px;
}

.recipe-square:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.25);
  border-color: rgba(255, 255, 255, 0.3);
  background: rgba(255, 255, 255, 0.12);
}

.preview-recipe-card {
  border: none !important;
  box-shadow: none !important;
  background: transparent !important;
  backdrop-filter: none !important;
  margin-left: -120px;
  margin-top: -60px;
  max-width: 260px;
}

.recipe-grid-scroll {
  display: grid;
  grid-template-columns: repeat(5, minmax(0, 1fr));
  gap: 0.5rem;
  max-height: calc(80px * 5 + 16px); /* 3 rows + gap */
  overflow-y: auto;
  padding-right: 6px; /* space for scrollbar */
}

.recipe-grid-scroll::-webkit-scrollbar {
  width: 8px;
  background: transparent;
}
.recipe-grid-scroll::-webkit-scrollbar-thumb {
  background: rgba(5, 134, 150, 0.25);
  border-radius: 8px;
  border: 2px solid rgba(255,255,255,0.15);
}
.recipe-grid-scroll::-webkit-scrollbar-thumb:hover {
  background: rgba(5, 134, 150, 0.45);
}

/* Firefox */
.recipe-grid-scroll {
  scrollbar-width: thin;
  scrollbar-color: rgba(5,134,150,0.25) rgba(255,255,255,0.05);
}
</style> 