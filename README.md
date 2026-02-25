import React, { useState, useEffect } from "react"; import { Card, CardContent } from "@/components/ui/card"; import { Button } from "@/components/ui/button"; import { Input } from "@/components/ui/input"; import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select"; import { CalendarDays, Clock, User, Scissors } from "lucide-react"; import { motion } from "framer-motion"; import { initializeApp } from "firebase/app"; import { getFirestore, collection, addDoc, onSnapshot, query, where } from "firebase/firestore";

// 🔥 CONFIGURE AQUI COM SEUS DADOS DO FIREBASE const firebaseConfig = { apiKey: "COLE_AQUI", authDomain: "COLE_AQUI", projectId: "Basspower.1", storageBucket: "COLE_AQUI", messagingSenderId: "COLE_AQUI", appId: "COLE_AQUI" };

const app = initializeApp(firebaseConfig); const db = getFirestore(app);

export default function MenezesBarbearia() { const [form, setForm] = useState({ nome: "", data: "", horario: "", servico: "" });

const [agendamentos, setAgendamentos] = useState([]); const [horarioOcupado, setHorarioOcupado] = useState(false);

useEffect(() => { const unsubscribe = onSnapshot(collection(db, "agendamentos"), (snapshot) => { const lista = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() })); setAgendamentos(lista); });

return () => unsubscribe();

}, []);

useEffect(() => { const ocupado = agendamentos.some( a => a.data === form.data && a.horario === form.horario ); setHorarioOcupado(ocupado); }, [form.data, form.horario, agendamentos]);

const handleSubmit = async (e) => { e.preventDefault();

if (horarioOcupado) {
  alert("Esse horário já está ocupado.");
  return;
}

await addDoc(collection(db, "agendamentos"), form);

alert("Agendamento confirmado!");
setForm({ nome: "", data: "", horario: "", servico: "" });

};

return ( <div className="min-h-screen bg-black text-white flex flex-col items-center p-6 gap-10"> <motion.div initial={{ opacity: 0, y: 40 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.6 }} className="w-full max-w-md" > <h1 className="text-3xl font-bold text-center mb-2"> Menezes Barbearia </h1> <p className="text-center text-zinc-400 mb-6"> Mairinque - SP </p>

<Card className="bg-zinc-900 rounded-2xl shadow-2xl">
      <CardContent className="p-6 space-y-4">
        <form onSubmit={handleSubmit} className="space-y-4">
          <div className="flex items-center gap-2">
            <User size={18} />
            <Input
              placeholder="Seu nome"
              value={form.nome}
              onChange={(e) => setForm({ ...form, nome: e.target.value })}
              required
              className="bg-zinc-800 border-none"
            />
          </div>

          <div className="flex items-center gap-2">
            <Scissors size={18} />
            <Select
              value={form.servico}
              onValueChange={(value) => setForm({ ...form, servico: value })}
            >
              <SelectTrigger className="bg-zinc-800 border-none">
                <SelectValue placeholder="Escolha o serviço" />
              </SelectTrigger>
              <SelectContent>
                <SelectItem value="Corte">Corte</SelectItem>
                <SelectItem value="Cabelo">Cabelo</SelectItem>
                <SelectItem value="Sobrancelha">Sobrancelha</SelectItem>
              </SelectContent>
            </Select>
          </div>

          <div className="flex items-center gap-2">
            <CalendarDays size={18} />
            <Input
              type="date"
              value={form.data}
              onChange={(e) => setForm({ ...form, data: e.target.value })}
              required
              className="bg-zinc-800 border-none"
            />
          </div>

          <div className="flex items-center gap-2">
            <Clock size={18} />
            <Input
              type="time"
              value={form.horario}
              onChange={(e) => setForm({ ...form, horario: e.target.value })}
              required
              className="bg-zinc-800 border-none"
            />
          </div>

          {horarioOcupado && (
            <p className="text-red-500 text-sm">
              Esse horário já está ocupado
            </p>
          )}

          <Button className="w-full rounded-2xl mt-4">
            Confirmar Agendamento
          </Button>
        </form>
      </CardContent>
    </Card>
  </motion.div>

  <div className="w-full max-w-2xl">
    <h2 className="text-xl font-semibold mb-4 text-center">
      Horários Agendados
    </h2>

    <div className="grid gap-4">
      {agendamentos.map((a) => (
        <Card key={a.id} className="bg-zinc-900 rounded-2xl">
          <CardContent className="p-4">
            <p className="font-semibold">{a.nome}</p>
            <p className="text-sm text-zinc-400">
              {a.servico} - {a.data} às {a.horario}
            </p>
          </CardContent>
        </Card>
      ))}
    </div>
  </div>
</div>

); }
