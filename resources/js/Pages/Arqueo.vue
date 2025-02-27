<script lang="ts">
import { defineComponent } from 'vue';
import Input from '../Components/input.vue';
import * as XLSX from 'xlsx';
import { Head } from '@inertiajs/vue3';

export default defineComponent({
    components: {
        Head,
        Input,
    },
    data() {
        return {
            saldoApertura: '0.00',
            saldoEfectivo: '0.00',
            descripcionDenominacionesApertura: '',
            descripcionDenominaciones: '',
            notas: [{ saldo: '0.00', lote: '' }],
            bancos: [{ saldo: '0.00', lote: '' }],
        };
    },
    methods: {
        updateSaldoApertura(newSaldo: string) {
            this.saldoApertura = newSaldo;
        },
        updateSaldoEfectivo(newSaldo: string) {
            this.saldoEfectivo = newSaldo;
        },
        updateDescripcionDenominacionesApertura(descripcion: string) {
            this.descripcionDenominacionesApertura = descripcion;
        },
        updateDescripcionDenominaciones(descripcion: string) {
            this.descripcionDenominaciones = descripcion;
        },
        updateSaldoBanco(index: number, newSaldo: string) {
            this.bancos[index].saldo = newSaldo;
        },
        addBanco() {
            this.bancos.push({ saldo: '0.00', lote: '' });
        },
        removeBanco(index: number) {
            if (this.bancos.length > 1) {
                this.bancos.splice(index, 1);
            }
        },
        clearInput(index: number) {
            this.bancos[index].lote = '';
        },
        updateSaldoNota(index: number, newSaldo: string) {
            this.notas[index].saldo = newSaldo;
        },
        addNota() {
            this.notas.push({ saldo: '0.00', lote: '' });
        },
        removeNota(index: number) {
            if (this.notas.length > 1) {
                this.notas.splice(index, 1);
            }
        },
        clearInputNota(index: number) {
            this.notas[index].lote = '';
        },
        downloadExcel() {
            const dataCuadre = [
                ['𝐃𝐞𝐬𝐜𝐫𝐢𝐩𝐜𝐢ó𝐧', '𝐌𝐨𝐧𝐭𝐨'],
                ['𝐄𝐟𝐞𝐜𝐭𝐢𝐯𝐨', `S/. ${this.saldoEfectivo}`],
                ['', ''],
                ['𝐁𝐚𝐧𝐜𝐨𝐬', '']
            ];

            this.bancos.forEach((banco, index) => {
                dataCuadre.push([`Banco ${index + 1} - Lote: ${banco.lote || 'No definido'}`, `S/. ${banco.saldo}`]);
            });
            dataCuadre.push(['Total Bancos', `S/. ${this.totalBancos}`]);

            dataCuadre.push(['', '']);
            dataCuadre.push(['𝐍𝐨𝐭𝐚𝐬 𝐝𝐞 𝐂𝐫é𝐝𝐢𝐭𝐨𝐬', '']);
            this.notas.forEach((nota, index) => {
                dataCuadre.push([`Nota Crédito ${index + 1} - Lote: ${nota.lote || 'No definido'}`, `S/. ${nota.saldo}`]);
            });
            dataCuadre.push(['Total Notas Créditos', `S/. ${this.totalNotas}`]);

            dataCuadre.push(['', '']);
            dataCuadre.push(['𝐓𝐨𝐭𝐚𝐥', `S/. ${this.total}`]);

            const wsCuadre = XLSX.utils.aoa_to_sheet(dataCuadre);

            const style = {
                font: { bold: true },
                fill: { fgColor: { rgb: 'C6EFCE' } },
            };

            const totalRowIndexes = [
                dataCuadre.findIndex(row => row[0] === 'Total Bancos'),
                dataCuadre.findIndex(row => row[0] === 'Total Notas Créditos'),
                dataCuadre.findIndex(row => row[0] === '𝐓𝐨𝐭𝐚𝐥'),
            ];
            totalRowIndexes.forEach(index => {
                const cellA = `A${index + 1}`;
                const cellB = `B${index + 1}`;
                if (wsCuadre[cellA]) wsCuadre[cellA].s = style;
                if (wsCuadre[cellB]) wsCuadre[cellB].s = style;
            });

            const descripcionData = [['𝐃𝐞𝐬𝐜𝐫𝐢𝐩𝐜𝐢ó𝐧 𝐝𝐞 𝐃𝐞𝐧𝐨𝐦𝐢𝐧𝐚𝐜𝐢𝐨𝐧𝐞𝐬 - 𝐄𝐟𝐞𝐜𝐭𝐢𝐯𝐨']];
            const descripcionRows = this.descripcionDenominaciones.split('\n').map(line => [line]);
            descripcionData.push(...descripcionRows);

            descripcionData.push(['', '']);
            descripcionData.push(['𝐃𝐞𝐬𝐜𝐫𝐢𝐩𝐜𝐢ó𝐧 𝐝𝐞 𝐃𝐞𝐧𝐨𝐦𝐢𝐧𝐚𝐜𝐢𝐨𝐧𝐞𝐬 - 𝐀𝐩𝐞𝐫𝐭𝐮𝐫𝐚']);
            const descripcionAperturaRows = this.descripcionDenominacionesApertura.split('\n').map(line => [line]);
            descripcionData.push(...descripcionAperturaRows);

            const wsDescripcion = XLSX.utils.aoa_to_sheet(descripcionData);

            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, wsCuadre, 'Cuadre de Caja');
            XLSX.utils.book_append_sheet(wb, wsDescripcion, 'Descripción de Efectivo');

            XLSX.writeFile(wb, 'cuadre_de_caja.xlsx');
        },
        reiniciar() {
            localStorage.removeItem('banco1');
            localStorage.removeItem('banco2');
            localStorage.removeItem('banco3');
            window.location.reload();
        },
    },
    computed: {
        totalBancos(): string {
            const total = this.bancos.reduce((sum: number, banco: any) => sum + parseFloat(banco.saldo), 0);
            return total.toFixed(2);
        },
        totalNotas(): string {
            const total = this.notas.reduce((sum: number, nota: any) => sum + parseFloat(nota.saldo), 0);
            return total.toFixed(2);
        },
        total(): string {
            const totalSaldo =
                parseFloat(this.saldoEfectivo) +
                this.bancos.reduce((sum: number, banco: any) => sum + parseFloat(banco.saldo), 0) +
                this.notas.reduce((sum: number, nota: any) => sum + parseFloat(nota.saldo), 0);
            return totalSaldo.toFixed(2);
        },
    },
});
</script>

<template>

    <Head title="Arqueo" />
    <div class="min-h-screen bg-[#181818] text-white flex flex-col gap-4 md:gap-8 relative">
        <div class="relative flex items-center gap-4">
            <img src="/images/blancokdosh.png" alt="Kdosh Store" class="ml-4 w-[50px] h-360px] md:w-[80px] md:h-[60px]">
            <div class="text-xl md:text-2xl font-bold mt-4 md:mt-6 ml-1 md:ml-6 mb-4 md:mb-6">
                CONTROL DE CUADRE DE CAJA
            </div>
            <div class="absolute top-full left-0 w-full border-b border-gray-400"></div>
        </div>

        <div class="flex justify-left ml-4 xs:ml-5 sm:ml-7 lg:ml-9 xl:ml-10">
            <button @click="reiniciar" class="bg-green-500 hover:bg-green-600 text-white py-2 px-4 rounded shadow-md">
                <i class="fa-solid fa-rotate"></i> Reiniciar
            </button>
        </div>

        <div class="bg-white mx-4 md:mx-9 rounded-md shadow-md">
            <div class="flex flex-col lg:flex-row gap-4 md:gap-6 p-4 md:p-8">
                <div class="bg-white text-[#181818] w-full lg:w-1/4 flex flex-col justify-center items-center">
                    <div class="w-full max-w-sm">
                        <span class="block font-semibold">Apertura</span>
                        <Input :showBillButton="true" @updateSaldo="updateSaldoApertura"
                            @updateDescripcion="updateDescripcionDenominacionesApertura" />
                    </div>
                    <div class="w-full max-w-sm">
                        <span class="block font-semibold">Efectivo</span>
                        <Input :showBillButton="true" @updateSaldo="updateSaldoEfectivo"
                            @updateDescripcion="updateDescripcionDenominaciones" />
                    </div>
                </div>

                <div
                    class="bg-[#181818] text-white p-4 md:p-6 rounded-md shadow-md flex flex-col lg:flex-row gap-6 xl:w-[1240px] w-full justify-center items-center">
                    <div class="flex-1">
                        <span class="block font-semibold mb-4">MÉTODOS DE PAGO</span>
                        <div class="flex flex-col gap-4 md:gap-6">
                            <span class="block font-semibold">BANCOS
                                <button @click="addBanco"
                                    class="bg-green-500 hover:bg-green-600 text-white py-[1.5px] px-[7.5px] rounded shadow-md ml-2">
                                    <i class="fa-solid fa-plus"></i>
                                </button>
                                <button @click="removeBanco(bancos.length - 1)"
                                    class="bg-red-500 hover:bg-red-600 text-white py-[1.5px] px-[7.5px] rounded shadow-md ml-2"
                                    :disabled="bancos.length <= 1">
                                    <i class="fa-solid fa-minus"></i>
                                </button>
                            </span>
                            <div class="border-l border-gray-500">
                                <div v-for="(banco, index) in bancos" :key="index" class="ml-5 mb-2">
                                    <span class="block font-semibold mb-2">{{ `Banco ${index + 1}` }}</span>
                                    <div class="flex">
                                        <Input :showBillButton="false" @updateSaldo="updateSaldoBanco(index, $event)" />
                                        <div class="relative w-30 ml-5">
                                            <input type="text"
                                                class="w-full p-2 focus:outline-none border border-gray-300 rounded appearance-none text-black"
                                                placeholder="N° de factura" v-model="banco.lote" />
                                            <i class="fas fa-times absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 cursor-pointer"
                                                @click="clearInput(index)"></i>
                                        </div>
                                    </div>
                                </div>
                            </div>
                            <span class="block font-semibold">NOTAS DE CRÉDITO
                                <button @click="addNota"
                                    class="bg-green-500 hover:bg-green-600 text-white py-[1.5px] px-[7.5px] rounded shadow-md ml-2">
                                    <i class="fa-solid fa-plus"></i>
                                </button>
                                <button @click="removeNota(notas.length - 1)"
                                    class="bg-red-500 hover:bg-red-600 text-white py-[1.5px] px-[7.5px] rounded shadow-md ml-2"
                                    :disabled="notas.length <= 1">
                                    <i class="fa-solid fa-minus"></i>
                                </button>
                            </span>
                            <div class="border-l border-gray-500">
                                <div v-for="(nota, index) in notas" :key="index" class="ml-5 mb-2">
                                    <span class="block font-semibold mb-2">{{ `Nota Crédito ${index + 1}` }}</span>
                                    <div class="flex">
                                        <Input :showBillButton="false" @updateSaldo="updateSaldoNota(index, $event)" />
                                        <div class="relative w-30 ml-5">
                                            <input type="text"
                                                class="w-full p-2 focus:outline-none border border-gray-300 rounded appearance-none text-black"
                                                placeholder="N° de factura" v-model="nota.lote" />
                                            <i class="fas fa-times absolute right-3 top-1/2 transform -translate-y-1/2 text-gray-400 cursor-pointer"
                                                @click="clearInputNota(index)"></i>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div class="flex-1 lg:ml-6">
                        <div class="flex justify-between font-bold text-sm mb-3">
                            <span>EFECTIVO</span>
                            <span>S/. {{ saldoEfectivo }}</span>
                        </div>

                        <span class="block font-bold mb-2 text-sm">BANCOS</span>
                        <ul class="mb-2 text-sm">
                            <div class="border-l border-gray-500">
                                <li v-for="(banco, index) in bancos" :key="index"
                                    class="flex justify-between ml-5 mb-3">
                                    <span><i class="fa-solid fa-plus mr-2"></i> {{ `Banco ${index + 1} - Lote:
                                        ${banco.lote || 'No definido'}` }}</span>
                                    <span>S/. {{ banco.saldo }}</span>
                                </li>
                                <li class="flex justify-between ml-5 font-bold mb-3">
                                    <span>TOTAL BANCOS</span>
                                    <span
                                        class="relative before:block before:absolute before:w-full before:h-[1.5px] before:bg-white before:top-[-3px]">
                                        S/. {{ totalBancos }}
                                    </span>
                                </li>
                            </div>
                        </ul>

                        <span class="block font-bold mb-2 text-sm">NOTA DE CRÉDITOS</span>
                        <ul class="mb-2 text-sm">
                            <div class="border-l border-gray-500">
                                <li v-for="(nota, index) in notas" :key="index" class="flex justify-between ml-5 mb-3">
                                    <span><i class="fa-solid fa-plus mr-2"></i> {{ `Nota C. ${index + 1} - Lote:
                                        ${nota.lote || 'No definido'}` }}</span>
                                    <span>S/. {{ nota.saldo }}</span>
                                </li>
                                <li class="flex justify-between ml-5 font-bold mb-3">
                                    <span>TOTAL NOTA CRÉDITOS</span>
                                    <span
                                        class="relative before:block before:absolute before:w-full before:h-[1.5px] before:bg-white before:top-[-3px]">
                                        S/. {{ totalNotas }}
                                    </span>
                                </li>
                            </div>
                        </ul>

                        <div class="flex justify-between font-bold text-sm">
                            <span class="ml-auto mr-4">TOTAL</span>
                            <span
                                class="relative before:block before:absolute before:w-full before:h-[1.5px] before:bg-white before:top-[-3px]">
                                S/. {{ total }}
                            </span>
                        </div>

                        <div class="flex justify-end mt-6 md:mt-10">
                            <button @click="downloadExcel"
                                class="bg-green-500 hover:bg-green-600 text-white py-2 px-4 rounded shadow flex items-center gap-2">
                                <i class="fas fa-file-excel text-white"></i>
                                Descargar
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
