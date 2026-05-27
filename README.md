import React, { useState } from "react";

const GOOGLE_SCRIPT_URL = "https://script.google.com/macros/s/AKfycbySgqc5Sgt_uNbmkirtz4HxwWLomzypcCHwhjyqSXxLL1d5KCnQtYwpS3CYgqDTUx4/exec";

export default function VotingPage() {
  const options = [
    { id: 1, title: "Opción 1" },
    { id: 2, title: "Opción 2" },
    { id: 3, title: "Opción 3" },
  ];

  const [name, setName] = useState("");
  const [selected, setSelected] = useState([]);
  const [submitted, setSubmitted] = useState(false);
  const [loading, setLoading] = useState(false);
  const [errorMessage, setErrorMessage] = useState("");

  const toggleOption = (id) => {
    setSelected((prev) => {
      if (prev.includes(id)) {
        return prev.filter((item) => item !== id);
      }

      return [...prev, id];
    });
  };

  const handleSubmit = async () => {
    setErrorMessage("");

    if (!name.trim()) {
      setErrorMessage("Por favor escribí tu nombre.");
      return;
    }

    if (selected.length === 0) {
      setErrorMessage("Seleccioná al menos una opción.");
      return;
    }

    setLoading(true);

    try {
      const selectedOptions = selected
        .map((id) => {
          const option = options.find((item) => item.id === id);
          return option ? option.title : "";
        })
        .join(", ");

      const params = new URLSearchParams({
        nombre: name,
        opciones: selectedOptions,
        fecha: new Date().toLocaleString(),
      });

      const requestUrl = `${GOOGLE_SCRIPT_URL}?${params.toString()}`;

      const form = document.createElement("form");

      form.method = "POST";
      form.action = GOOGLE_SCRIPT_URL;
      form.target = "hidden_iframe";
      form.style.display = "none";

      const fields = {
        nombre: name,
        opciones: selectedOptions,
        fecha: new Date().toLocaleString(),
      };

      Object.entries(fields).forEach(([key, value]) => {
        const input = document.createElement("input");
        input.type = "hidden";
        input.name = key;
        input.value = value;
        form.appendChild(input);
      });

      let iframe = document.getElementById("hidden_iframe");

      if (!iframe) {
        iframe = document.createElement("iframe");
        iframe.id = "hidden_iframe";
        iframe.name = "hidden_iframe";
        iframe.style.display = "none";
        document.body.appendChild(iframe);
      }

      document.body.appendChild(form);
      form.submit();
      document.body.removeChild(form);

      await new Promise((resolve) => setTimeout(resolve, 1000));

      setSubmitted(true);
    } catch (error) {
      console.error(error);
      setErrorMessage(
        "No se pudo enviar el voto. Revisá que el Apps Script esté publicado como aplicación web pública."
      );
    } finally {
      setLoading(false);
    }
  };

  const resetForm = () => {
    setName("");
    setSelected([]);
    setSubmitted(false);
    setErrorMessage("");
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-slate-100 via-slate-200 to-slate-300 flex items-center justify-center p-6">
      <div className="w-full max-w-2xl rounded-3xl bg-white p-8 shadow-2xl">
        <div className="mb-8 text-center">
          <h1 className="mb-3 text-4xl font-bold">
            🗳️ Sistema de Votación
          </h1>

          <p className="text-lg text-slate-600">
            Elegí una o más opciones y completá tu nombre.
          </p>
        </div>

        {!submitted ? (
          <>
            <div className="mb-6">
              <label className="mb-2 block text-lg font-semibold">
                Nombre de la persona
              </label>

              <input
                type="text"
                value={name}
                onChange={(e) => setName(e.target.value)}
                placeholder="Escribí tu nombre..."
                className="w-full rounded-2xl border border-slate-300 p-4 focus:outline-none focus:ring-2 focus:ring-black"
              />
            </div>

            <div className="mb-8 grid grid-cols-1 gap-4">
              {options.map((option) => {
                const isSelected = selected.includes(option.id);

                return (
                  <button
                    key={option.id}
                    type="button"
                    onClick={() => toggleOption(option.id)}
                    className={`rounded-2xl border-2 p-5 text-left transition-all duration-300 hover:scale-[1.02] ${
                      isSelected
                        ? "border-black bg-black text-white shadow-lg"
                        : "border-slate-200 bg-slate-50 hover:border-slate-400"
                    }`}
                  >
                    <div className="flex items-center justify-between">
                      <span className="text-xl font-semibold">
                        {option.title}
                      </span>

                      {isSelected ? (
                        <span className="text-2xl">✔</span>
                      ) : null}
                    </div>
                  </button>
                );
              })}
            </div>

            {errorMessage ? (
              <div className="mb-6 rounded-2xl border border-red-300 bg-red-100 p-4 text-center text-red-700">
                {errorMessage}
              </div>
            ) : null}

            <div className="flex justify-center">
              <button
                type="button"
                onClick={handleSubmit}
                disabled={loading}
                className="rounded-2xl bg-black px-8 py-4 text-lg font-semibold text-white transition hover:opacity-90 disabled:cursor-not-allowed disabled:opacity-50"
              >
                {loading ? "Enviando..." : "Enviar voto"}
              </button>
            </div>
          </>
        ) : (
          <div className="py-10 text-center">
            <div className="mb-4 text-6xl">✅</div>

            <h2 className="mb-3 text-3xl font-bold">
              ¡Gracias por votar!
            </h2>

            <p className="mb-6 text-lg text-slate-600">
              Tu voto fue enviado correctamente.
            </p>

            <div className="mx-auto max-w-md rounded-2xl bg-slate-100 p-5 text-left">
              <p className="mb-2">
                <span className="font-bold">Nombre:</span> {name}
              </p>

              <p>
                <span className="font-bold">Opciones elegidas:</span>{" "}
                {selected
                  .map((id) => {
                    const option = options.find((item) => item.id === id);
                    return option ? option.title : "";
                  })
                  .join(", ")}
              </p>
            </div>

            <button
              type="button"
              onClick={resetForm}
              className="mt-8 rounded-2xl border border-slate-300 px-6 py-3 font-semibold transition hover:bg-slate-100"
            >
              Nuevo voto
            </button>
          </div>
        )}
      </div>
    </div>
  );
}
