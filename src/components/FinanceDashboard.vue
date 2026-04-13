<script setup>
import { ref, computed } from 'vue';
import { Icon } from '@iconify/vue';

const transactions = ref([
    {
        id: 1,
        date: '2026-04-01',
        desc: 'Salário',
        amount: 10000,
        type: 'income',
        category: 'Salário'
    },
    {
        id: 2,
        date: '2026-04-03',
        desc: 'Feira do Mês',
        amount: -800,
        type: 'expense',
        category: 'Supermercado'
    },
    {
        id: 3,
        date: '2026-04-03',
        desc: 'Energia Elétrica',
        amount: -60,
        type: 'expense',
        category: 'Moradia'
    },
    {
        id: 4,
        date: '2026-04-03',
        desc: 'Aluguel',
        amount: -650,
        type: 'expense',
        category: 'Moradia'
    },
    {
        id: 5,
        date: '2026-04-05',
        desc: 'Condomínio',
        amount: -200,
        type: 'expense',
        category: 'Moradia'
    },
    {
        id: 6,
        date: '2026-04-05',
        desc: 'Internet',
        amount: -110,
        type: 'expense',
        category: 'Moradia'
    },
    {
        id: 7,
        date: '2026-04-05',
        desc: 'Crédito do Celular (Jhonata)',
        amount: -30,
        type: 'expense',
        category: 'Comunicação'
    },
    {
        id: 8,
        date: '2026-04-05',
        desc: 'Crédito do Celular (Ray)',
        amount: -40,
        type: 'expense',
        category: 'Comunicação'
    },
    {
        id: 9,
        date: '2026-04-05',
        desc: 'Plano Odontológico',
        amount: -60,
        type: 'expense',
        category: 'Saúde'
    },
    {
        id: 10,
        date: '2026-04-05',
        desc: 'Manutenção do Aparelho',
        amount: -100,
        type: 'expense',
        category: 'Saúde'
    },
    {
        id: 11,
        date: '2026-04-05',
        desc: 'Advogada',
        amount: -300,
        type: 'expense',
        category: 'Imprevistos'
    },
    {
        id: 12,
        date: '2026-04-05',
        desc: 'Empréstimo',
        amount: -790,
        type: 'expense',
        category: 'Imprevistos'
    },
    {
        id: 12,
        date: '2026-04-05',
        desc: 'Fatura',
        amount: -210,
        type: 'expense',
        category: 'Cartão de Crédito'
    }
]);

const newDesc = ref('');
const newAmount = ref('');
const newType = ref('expense');
const newCategory = ref('');

const expenseCategoryFilter = ref('');
const incomeCategoryFilter = ref('');
const expenseCategories = computed(() => [
    ...new Set(
        transactions.value
            .filter((t) => t.type === 'expense')
            .map((t) => t.category)
    )
]);
const incomeCategories = computed(() => [
    ...new Set(
        transactions.value
            .filter((t) => t.type === 'income')
            .map((t) => t.category)
    )
]);
const expenses = computed(() =>
    transactions.value.filter(
        (t) =>
            t.type === 'expense' &&
            (!expenseCategoryFilter.value ||
                t.category === expenseCategoryFilter.value)
    )
);
const incomes = computed(() =>
    transactions.value.filter(
        (t) =>
            t.type === 'income' &&
            (!incomeCategoryFilter.value ||
                t.category === incomeCategoryFilter.value)
    )
);

const balance = computed(() =>
    transactions.value.reduce((s, t) => s + Number(t.amount), 0)
);

const suggestedCategories = computed(() => {
    return newType.value === 'income' ? incomeCategories.value : expenseCategories.value;
});


function formatDateBR(dateStr) {
    // expects yyyy-mm-dd, returns dd/mm/yyyy
    if (!dateStr) return '';
    const [y, m, d] = dateStr.split('-');
    return `${d}/${m}/${y}`;
}

function formatBRL(value) {
    return new Intl.NumberFormat('pt-BR', { style: 'currency', currency: 'BRL' }).format(value);
}

function addTransaction() {
    if (!newDesc.value || !newAmount.value || !newCategory.value) return;
    const amount =
        Number(newAmount.value) * (newType.value === 'income' ? 1 : -1);
    transactions.value.unshift({
        id: Date.now(),
        date: new Date().toISOString().slice(0, 10),
        desc: newDesc.value,
        amount,
        type: newType.value,
        category: newCategory.value
    });
    newDesc.value = '';
    newAmount.value = '';
    newCategory.value = '';
}

function removeTransaction(id) {
    transactions.value = transactions.value.filter((t) => t.id !== id);
}
</script>

<template>
    <section class="p-3 max-w-6xl mx-auto">
        <header class="flex items-center justify-between mb-6">
            <div class="flex items-center justify-center gap-2 text-white bg-black px-4 py-2 rounded-lg shadow-sm">
                <Icon icon="ic:baseline-monitor-heart" class="w-6 h-6" />
                <h1 class="text-lg font-bold">
                    Fine
                </h1>
            </div>
            <div class="text-right border px-4 py-2 rounded-lg shadow-sm"
                :class="balance > 0 ? 'bg-green-100 border-green-300 text-green-500' : 'bg-red-100 border-red-300 text-red-500'">
                <div class="text-sm">Saldo</div>
                <div class="text-xl font-bold">
                    <span class="inline-flex items-center gap-2">
                        {{ formatBRL(balance) }}
                    </span>
                </div>
            </div>
        </header>

        <div class="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
            <div class="md:col-span-2">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <!-- Expenses Column -->
                    <div class="bg-white p-4 rounded-lg shadow-sm">
                        <div class="flex items-center justify-between mb-3">
                            <h2 class="font-medium text-red-500">Despesas</h2>
                            <select v-model="expenseCategoryFilter" class="text-xs px-2 py-1 border rounded">
                                <option value="">Todas</option>
                                <option v-for="cat in expenseCategories" :key="cat" :value="cat">
                                    {{ cat }}
                                </option>
                            </select>
                        </div>
                        <ul class="flex flex-col gap-2 overflow-y-auto min-h-100 max-h-100">
                            <li v-for="tx in expenses" :key="tx.id"
                                class="p-3 bg-red-50 rounded border border-red-100 text-sm">
                                <div class="flex items-center justify-between">
                                    <div>
                                        <div>
                                            {{ tx.desc }}
                                        </div>
                                        <div>
                                            {{ formatDateBR(tx.date) }}
                                        </div>
                                    </div>
                                    <div class="flex items-center gap-2">
                                        <div class="text-white bg-red-400 px-2 py-1 rounded">
                                            {{ formatBRL(tx.amount) }}
                                        </div>
                                    </div>
                                </div>
                                <div class="flex items-center justify-between mt-2">
                                    <p class="px-2 py-1 rounded text-xs text-white w-fit bg-indigo-400">{{ tx.category
                                    }}</p>
                                    <button @click="removeTransaction(tx.id)"
                                        class="hover:text-red-400 active:text-red-400">
                                        <Icon icon="lucide:trash-2" class="w-5 h-5" />
                                    </button>
                                </div>
                            </li>
                        </ul>
                        <p
                            class="mt-3 bg-red-50 border border-red-100 text-red-500 px-3 py-2 rounded text-sm font-medium">
                            {{formatBRL(expenses.reduce((s, t) => s + Number(t.amount), 0))}}
                        </p>
                    </div>
                    <!-- Incomes Column -->
                    <div class="bg-white p-4 rounded-lg shadow-sm">
                        <div class="flex items-center justify-between mb-3">
                            <h2 class="font-medium text-green-700">Receitas</h2>
                            <select v-model="incomeCategoryFilter" class="text-xs px-2 py-1 border rounded">
                                <option value="">Todas</option>
                                <option v-for="cat in incomeCategories" :key="cat" :value="cat">
                                    {{ cat }}
                                </option>
                            </select>
                        </div>
                        <ul class="flex flex-col gap-2 overflow-y-auto min-h-100 max-h-100">
                            <li v-for="tx in incomes" :key="tx.id"
                                class="p-3 bg-green-50 rounded text-sm border border-green-100">
                                <div class="flex items-center justify-between">
                                    <div>
                                        <div>
                                            {{ tx.desc }}
                                        </div>
                                        <div>
                                            {{ formatDateBR(tx.date) }}
                                        </div>
                                    </div>
                                    <div class="flex items-center gap-4">
                                        <div class="text-white bg-green-500 px-2 py-1 rounded">
                                            {{ formatBRL(tx.amount) }}
                                        </div>
                                    </div>
                                </div>
                                <div class="flex items-center justify-between mt-2">
                                    <p class="px-2 py-1 rounded text-xs text-white w-fit bg-indigo-400">{{ tx.category
                                        }}</p>
                                    <button @click="removeTransaction(tx.id)"
                                        class="hover:text-red-400 active:text-red-400">
                                        <Icon icon="lucide:trash-2" class="w-5 h-5" />
                                    </button>
                                </div>
                            </li>
                        </ul>
                        <p
                            class="mt-3 bg-green-50 border border-green-100 text-green-500 px-3 py-2 rounded text-sm font-medium">
                            {{formatBRL(incomes.reduce((s, t) => s + Number(t.amount), 0))}}
                        </p>
                    </div>
                </div>
            </div>
            <aside class="bg-white p-4 rounded-lg shadow-sm">
                <h3 class="font-medium text-slate-700 mb-2">
                    Adicionar Lançamento
                </h3>
                <div class="flex flex-col gap-2">
                    <input v-model="newDesc" placeholder="Descrição" class="px-3 py-2 border rounded" />
                    <input v-model="newAmount" placeholder="Valor" type="number" class="px-3 py-2 border rounded" />
                    <div class="relative">
                        <select v-model="newCategory" class="px-3 py-2 border rounded w-full bg-white mb-1">
                            <option value="" disabled selected hidden>
                                Categoria
                            </option>
                            <option v-for="cat in suggestedCategories" :key="cat" :value="cat">
                                {{ cat }}
                            </option>
                            <option v-if="
                                newCategory &&
                                !suggestedCategories.includes(newCategory)
                            " :value="newCategory">
                                {{ newCategory }}
                            </option>
                        </select>
                        <input v-if="
                            !suggestedCategories.includes(newCategory) &&
                            newCategory !== ''
                        " v-model="newCategory" placeholder="Nova categoria"
                            class="absolute top-0 left-0 px-3 py-2 border rounded w-full bg-transparent focus:bg-white"
                            style="pointer-events: auto" />
                    </div>
                    <select v-model="newType" class="px-3 py-2 border rounded">
                        <option value="income">Receita</option>
                        <option value="expense">Despesa</option>
                    </select>
                    <button @click="addTransaction"
                        class="mt-2 bg-black text-white px-3 py-2 rounded hover:bg-neutral-800 active:bg-gray-800">
                        Adicionar
                    </button>
                </div>
            </aside>
        </div>
    </section>
</template>
