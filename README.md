# Parcelable

Por que usar o Parcelable ao invés do Serializable
Começando com o Serializable
Na hora da passagem de parâmetros entre Activitys nos desenvolvedores enfrentamos um problema serio de desempenho. Ao passar os parâmetros/valores diretamente usando o Bundle o consumo de memoria do aparelho aumenta de forma considerável, já que os objetos são passados de forma “bruta”, isso leva um processamento maior na hora de carregar a Activity e todas as suas Views.

Vendo esse problema, foi criado o método Serializable que faz com que o objeto seja transformado em binário (serializando), diminuindo seu tamanho para passar com mais facilidade. Chegando no outro lado o objeto vai ser transformado de binário para o objeto que era antes (descerialização). Pode parecer que aumentou o número de processos, pode até ser, mas agora o objeto ficou muito menor na hora de passar no gargalo de uma Activity para outra. Veja a imagem abaixo para perceber como funciona:


Veja que é mais fácil carregar uma boia seca do que uma cheia … eu sei, péssimo exemplo :( …

Agora vamos aos códigos. Para você usar o Serializable é preciso implementar na sua classe a interface Serializable, veja:

import java.io.Serializable;
public class Pessoa implements Serializable{

    private int codigo;
    private String nome;
    public static final long  serialVersionUID = 100L;

    public int getCodigo() {
        return codigo;
    }
    public void setCodigo(int codigo) {
        this.codigo = codigo;
    }
    public String getNome() {
        return nome;
    }
    public void setNome(String nome) {
        this.nome = nome;
    }
}
Veja que é preciso usar apenas o implements Serializable na sua classe e tudo de boas.

Podendo usar a passagem de objetos mais complexos (ArrayList nesse caso), veja:

ArrayList<Pessoa> pessoas = new ArrayList<Pessoa>();
pessoas.add(new Pessoa(1, "Lucas"));
pessoas.add(new Pessoa(2, "Jose"));

Intent it = new Intent(this, Tela2Activity.class);
// Caso queira passar a lista toda use
it.putExtra("pessoas", pessoas); 
// Caso queira passar apenas um objeto
it.putExtra("pessoa", pessoas.get(0)); 
startActivity(it);
Vamos ao Parcelable
Viu? Simples. Mas como o Google consegue sempre mais existe uma forma ainda mais rápida de passagem de parâmetros, a toda poderosa Parcelable.

Parcelable segundo a documentação oficial:

Interface for classes whose instances can be written to and restored from a Parcel. Classes implementing the Parcelable interface must also have a non-null static field called CREATOR of a type that implements the Parcelable.Creator interface.

Trocando em miúdos, Parcelable faz o que o Serializable faz, só que melhor. E isso saiu de graça? Não, a implementação do Parcelable é mais densa, mas não é complicada, só trabalhosa, vamos aos códigos:

import android.os.Parcel;
import android.os.Parcelable;

public class Cliente implements Parcelable {
 private int codigo;
 private String nome;

 public Cliente(int codigo, String nome) {
   this.codigo = codigo;
   this.nome = nome;
 }

 private Cliente(Parcel p){
   codigo = from.readInt();
   nome = from.readString();
 }

 public static final Parcelable.Creator<Cliente>
   CREATOR = new Parcelable.Creator<Cliente>() {

   public Cliente createFromParcel(Parcel in) {
     return new Cliente(in);
   }

   public Cliente[] newArray(int size) {
     return new Cliente[size];
   }
 };

 public int getCodigo() {
   return codigo;
 }

 public void setCodigo(int codigo) {
   this.codigo = codigo;
 }

 public String getNome() {
   return nome;
 }

 public void setNome(String nome) {
   this.nome = nome;
 }

 @Override
 public int describeContents() {
   return 0;
 }

 @Override
 public void writeToParcel(Parcel dest, int flags) {
   dest.writeInt(codigo);
   dest.writeString(nome); 
 }
}
Veja que, ao implementar o Parcelable você precisa sobrescrever dois metodos describeContents e writeToParcel, além de precisar criar um construtor que recebe o objeto Parcel e outro construtor que recebe e seta os próprios paramentos da classe/objeto. Você só precisa implementar o método writeToParcel, ele faz o objeto Parcel (dest) receber todos os atributos da classe, o flags fica para outra postagem. Dependendo do tipo do objeto você vai escrever (write) o objeto Parcel de forma diferente, veja:

dest.writeInt(codigo); //codigo é inteiro
dest.writeString(nome); //nome é String
dest.writeFloat(preco); //preco é float
Como existem diversos tipos de escrita para diversos objetos, primitivos ou não primitivos, vou deixar o link para visualizar todos.

DOCUMENTAÇÃO DO PARCEL

Shooow! Agora vamos vamos montar o objeto dentro das Activitys:

Cliente cliente = new Cliente(1, "Glauber");
Intent it = new Intent(this, Teste2Activity.class);
it.putExtra("cliente", cliente);
startActivity(it);
Massa! Agora, dentro da segunda Activity (Teste2Activity) você pode usar o seguinte método:

Cliente c =  getIntent().getExtras().getParcelable("cliente");
Simples, rápido, fácil …

Só para se ter uma noção do quanto o Parcelable é rápido, em relação ao Serializable, veja esse teste desempenho realizado:


A fonte do teste segue nesse link.

Um adendo: se sua aplicação for pequena (duas telas e sem objetos complexos) não vejo vantagem em usar nenhuma desses dois métodos, mas vale fazer para incentivar o uso de boas práticas.
