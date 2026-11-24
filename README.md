using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Text;

namespace LocadoraCarros
{
    public class Carro
    {
        public string Placa { get; set; }
        public string Modelo { get; set; }
        public string Categoria { get; set; }
        public int Ano { get; set; }
        public double Diaria { get; set; }
        public bool Disponivel { get; set; } = true;

        public Carro(string placa, string modelo, string categoria, int ano, double diaria)
        {
            Placa = placa;
            Modelo = modelo;
            Categoria = categoria;
            Ano = ano;
            Diaria = diaria;
        }

        public override string ToString()
        {
            string status = Disponivel ? "Disponível" : "Alugado";
            return $"{Placa} | {Modelo,-14} | {Categoria,-10} | {Ano} | R$ {Diaria,7:F2} | {status}";
        }
    }

    public static class Algoritmos
    {
        public static List<Carro> InsertionSort(List<Carro> lista, Func<Carro, IComparable> chave)
        {
            var v = new List<Carro>(lista);
            for (int i = 1; i < v.Count; i++)
            {
                Carro atual = v[i];
                int j = i - 1;
                while (j >= 0 && chave(v[j]).CompareTo(chave(atual)) > 0)
                {
                    v[j + 1] = v[j];
                    j--;
                }
                v[j + 1] = atual;
            }
            return v;
        }

        public static List<Carro> MergeSort(List<Carro> lista, Func<Carro, IComparable> chave)
        {
            if (lista.Count <= 1) return new List<Carro>(lista);

            int meio = lista.Count / 2;
            var esq = MergeSort(lista.GetRange(0, meio), chave);
            var dir = MergeSort(lista.GetRange(meio, lista.Count - meio), chave);

            var res = new List<Carro>();
            int i = 0, j = 0;
            while (i < esq.Count && j < dir.Count)
            {
                if (chave(esq[i]).CompareTo(chave(dir[j])) <= 0) res.Add(esq[i++]);
                else res.Add(dir[j++]);
            }
            while (i < esq.Count) res.Add(esq[i++]);
            while (j < dir.Count) res.Add(dir[j++]);
            return res;
        }

        public static List<Carro> BuscaLinearModelo(List<Carro> lista, string termo)
        {
            var resultado = new List<Carro>();
            foreach (var c in lista)
            {
                if (c.Modelo.Contains(termo, StringComparison.OrdinalIgnoreCase))
                    resultado.Add(c);
            }
            return resultado;
        }

        public static Carro? BuscaBinariaPlaca(List<Carro> lista, string placa)
        {
            var ordenada = MergeSort(lista, c => c.Placa);
            int ini = 0, fim = ordenada.Count - 1;
            while (ini <= fim)
            {
                int meio = (ini + fim) / 2;
                int cmp = string.CompareOrdinal(ordenada[meio].Placa, placa);
                if (cmp == 0) return ordenada[meio];
                if (cmp < 0) ini = meio + 1;
                else fim = meio - 1;
            }
            return null;
        }

        public static List<Carro> SelecionarPorCategoria(List<Carro> lista, string categoria)
        {
            var resultado = new List<Carro>();
            foreach (var c in lista)
            {
                if (c.Disponivel && c.Categoria.Equals(categoria, StringComparison.OrdinalIgnoreCase))
                    resultado.Add(c);
            }
            return resultado;
        }

        public static List<Carro> SelecionarPorPrecoMax(List<Carro> lista, double precoMax)
        {
            var resultado = new List<Carro>();
            foreach (var c in lista)
            {
                if (c.Disponivel && c.Diaria <= precoMax)
                    resultado.Add(c);
            }
            return resultado;
        }
    }

    class Program
    {
        static void Mostrar(List<Carro> lista)
        {
            if (lista.Count == 0)
            {
                Console.WriteLine("  Nenhum carro encontrado.");
                return;
            }
            foreach (var c in lista) Console.WriteLine("  " + c);
        }

        static void AlterarStatus(List<Carro> frota, string placa, bool disponivel)
        {
            var carro = Algoritmos.BuscaBinariaPlaca(frota, placa);
            if (carro == null) Console.WriteLine("  Placa não encontrada.");
            else if (carro.Disponivel == disponivel) Console.WriteLine("  O carro já está nesse status.");
            else
            {
                carro.Disponivel = disponivel;
                Console.WriteLine(disponivel ? "  Carro devolvido!" : "  Carro alugado!");
            }
        }

        static List<Carro> FrotaInicial() => new List<Carro>
        {
            new Carro("ABC1D23", "Onix",    "Econômico", 2022, 120.0),
            new Carro("DEF4G56", "HB20",    "Econômico", 2021, 110.0),
            new Carro("GHI7J89", "Corolla", "Sedan",     2023, 250.0),
            new Carro("JKL0M12", "Civic",   "Sedan",     2020, 230.0),
            new Carro("MNO3P45", "Compass", "SUV",       2022, 340.0),
            new Carro("PQR6S78", "Creta",   "SUV",       2023, 310.0),
            new Carro("STU9V01", "Hilux",   "Picape",    2021, 400.0),
            new Carro("VWX2Y34", "Mobi",    "Econômico", 2019, 90.0),
        };

        static void Main()
        {
            Console.OutputEncoding = Encoding.UTF8;
            var frota = FrotaInicial();

            while (true)
            {
                Console.WriteLine("\n===== LOCADORA DE CARROS =====");
                Console.WriteLine("1. Listar frota");
                Console.WriteLine("2. Ordenar por preço (Insertion Sort)");
                Console.WriteLine("3. Ordenar por ano (Merge Sort)");
                Console.WriteLine("4. Buscar por modelo (Busca Linear)");
                Console.WriteLine("5. Buscar por placa (Busca Binária)");
                Console.WriteLine("6. Selecionar por categoria");
                Console.WriteLine("7. Selecionar por preço máximo");
                Console.WriteLine("8. Alugar carro");
                Console.WriteLine("9. Devolver carro");
                Console.WriteLine("0. Sair");
                Console.Write("Opção: ");
                string op = (Console.ReadLine() ?? "").Trim();

                switch (op)
                {
                    case "1":
                        Mostrar(frota);
                        break;
                    case "2":
                        Mostrar(Algoritmos.InsertionSort(frota, c => c.Diaria));
                        break;
                    case "3":
                        Mostrar(Algoritmos.MergeSort(frota, c => c.Ano));
                        break;
                    case "4":
                        Console.Write("Modelo: ");
                        Mostrar(Algoritmos.BuscaLinearModelo(frota, Console.ReadLine() ?? ""));
                        break;
                    case "5":
                        Console.Write("Placa: ");
                        var c = Algoritmos.BuscaBinariaPlaca(frota, (Console.ReadLine() ?? "").ToUpper());
                        Mostrar(c != null ? new List<Carro> { c } : new List<Carro>());
                        break;
                    case "6":
                        Console.Write("Categoria: ");
                        Mostrar(Algoritmos.SelecionarPorCategoria(frota, Console.ReadLine() ?? ""));
                        break;
                    case "7":
                        Console.Write("Preço máximo: ");
                        string entrada = (Console.ReadLine() ?? "").Replace(',', '.');
                        if (double.TryParse(entrada, NumberStyles.Any, CultureInfo.InvariantCulture, out double max))
                            Mostrar(Algoritmos.SelecionarPorPrecoMax(frota, max));
                        else
                            Console.WriteLine("  Valor inválido.");
                        break;
                    case "8":
                        Console.Write("Placa: ");
                        AlterarStatus(frota, (Console.ReadLine() ?? "").ToUpper(), false);
                        break;
                    case "9":
                        Console.Write("Placa: ");
                        AlterarStatus(frota, (Console.ReadLine() ?? "").ToUpper(), true);
                        break;
                    case "0":
                        Console.WriteLine("Encerrando...");
                        return;
                    default:
                        Console.WriteLine("  Opção inválida.");
                        break;
                }
            }
        }
    }
}  
