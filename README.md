- 👋 Hi, I’m @Eddys81
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Eddys81/Eddys81 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
"use client";
import React from "react";
 
function MainComponent() {
  const { useState, useEffect, useCallback } = React;
 
  const [isAuthenticated, setIsAuthenticated] = useState(() => {
	const auth = localStorage.getItem("auth");
	return auth ? JSON.parse(auth) : null;
  });
 
  const [loginData, setLoginData] = useState({
	username: "",
	password: "",
  });
 
  const users = {
	critico01: {
  	password: "Cr!t1c0@2024",
  	role: "operator",
  	sector: "Área Crítica",
	},
	lavagem01: {
  	password: "Lav@g3m2024!",
  	role: "operator",
  	sector: "Lavagem",
	},
	centrifuga01: {
  	password: "C3ntr!2024#",
  	role: "operator",
  	sector: "Centrífuga",
	},
	secador01: {
  	password: "S3cad0r@2024",
  	role: "operator",
  	sector: "Secador",
	},
	dobra01: {
  	password: "D0br4!2024#",
  	role: "operator",
  	sector: "Dobra",
	},
	calandra01: {
  	password: "Cal@ndra#2024",
  	role: "operator",
  	sector: "Calandra",
	},
	expedicao01: {
  	password: "Exp3d!c@o2024",
  	role: "operator",
  	sector: "Expedição",
	},
	entrega01: {
  	password: "Entreg@2024!",
  	role: "operator",
  	sector: "Entrega Concluída",
	},
	admin: {
  	password: "Adm1n#2024!",
  	role: "admin",
  	sector: "all",
	},
  };
 
  const handleLogin = (e) => {
    e.preventDefault();
	const user = users[loginData.username];
 
	if (user && user.password === loginData.password) {
  	const authData = {
    	username: loginData.username,
    	role: user.role,
    	sector: user.sector,
  	};
      localStorage.setItem("auth", JSON.stringify(authData));
      setIsAuthenticated(authData);
	} else {
      alert("Usuário ou senha inválidos");
	}
  };
 
  useEffect(() => {
	const auth = localStorage.getItem("auth");
	if (auth) {
      setIsAuthenticated(JSON.parse(auth));
	}
  }, []);
 
  const handleLogout = () => {
	localStorage.removeItem("auth");
    setIsAuthenticated(null);
  };
 
  const filterRowsByUserAccess = (rows) => {
	if (!isAuthenticated) return [];
 
	let filteredRows =
      isAuthenticated.role === "admin"
    	? rows
    	: rows.filter((row) => row.sector === isAuthenticated.sector);
 
	return filteredRows.sort((a, b) => {
  	const dateA = new Date(a.date);
  	const dateB = new Date(b.date);
 
  	if (dateA > dateB) return -1;
  	if (dateA < dateB) return 1;
 
  	if (a.startTime && b.startTime) {
    	const timeA = new Date(`1970-01-01T${a.startTime}`);
    	const timeB = new Date(`1970-01-01T${b.startTime}`);
    	return timeB - timeA;
  	}
  	return 0;
	});
  };
 
  const canAccessFeature = (feature) => {
	if (!isAuthenticated) return false;
	if (isAuthenticated.role === "admin") return true;
 
	const operatorFeatures = ["table", "import", "export"];
	return (
      isAuthenticated.role === "operator" && operatorFeatures.includes(feature)
	);
  };
 
  const [rows, setRows] = useState(() => {
	const savedRows = localStorage.getItem("laundryRows");
	const parsedRows = savedRows ? JSON.parse(savedRows) : [];
 
	return parsedRows.sort((a, b) => {
  	const dateA = new Date(a.date);
  	const dateB = new Date(b.date);
 
  	if (dateA > dateB) return -1;
  	if (dateA < dateB) return 1;
 
  	if (a.startTime && b.startTime) {
    	const timeA = new Date(`1970-01-01T${a.startTime}`);
    	const timeB = new Date(`1970-01-01T${b.startTime}`);
    	return timeB - timeA;
  	}
  	return 0;
	});
  });
 
  const [editingRow, setEditingRow] = useState(null);
  const [showForm, setShowForm] = useState(false);
  const [showDashboard, setShowDashboard] = useState(true);
  const [dateFilter, setDateFilter] = useState({
	startDate: new Date().toISOString().split("T")[0],
	endDate: new Date().toISOString().split("T")[0],
  });
  const [sectorStats, setSectorStats] = useState({});
 
  const sectors = [
	"Área Crítica",
    "Lavagem",
    "Centrífuga",
    "Secador",
	"Dobra",
    "Calandra",
    "Expedição",
	"Entrega Concluída",
  ];
  const clients = [
    "Maracanaú",
	"IVV",
    "Pindoretama",
    "Quixadá",
    "Batista",
    "Pacatuba",
  ];
  const clothingTypes = [
    "Lençol",
    "Toalha",
    "Fardamento",
	"Fardamento Tactel",
	"Campo cirúrgico",
    "Travessa",
  ];
  const processTypes = [
	"Leve",
    "Pesado",
	"Super Pesado",
    "Retorno",
	"Campo Cirúrgico",
    "Fardamento",
  ];
 
  const initialForm = {
	date: new Date().toISOString().split("T")[0],
	startTime: "",
	endTime: "",
	batchId: "",
	client: "",
	clothingType: "",
	totalWeight: "",
	processedWeight: "",
	pieces: "",
	processType: "",
	sector: sectors[0],
	status: "pending",
  };
 
  const [formData, setFormData] = useState(initialForm);
 
  useEffect(() => {
    localStorage.setItem("laundryRows", JSON.stringify(rows));
    setSectorStats(calculateSectorStats());
  }, [rows, dateFilter]);
 
  const handleSubmit = (e) => {
    e.preventDefault();
	if (editingRow) {
  	setRows(
    	rows.map((row) =>
      	row.id === editingRow ? { ...formData, id: editingRow } : row
    	)
  	);
	} else {
  	setRows([
    	...rows,
    	{
      	...formData,
      	id: Date.now(),
      	sector:
            isAuthenticated.role === "admin"
          	? sectors[0]
          	: isAuthenticated.sector,
      	startTime: new Date().toLocaleTimeString("pt-BR", {
        	hour: "2-digit",
        	minute: "2-digit",
      	}),
    	},
  	]);
	}
    setFormData(initialForm);
    setEditingRow(null);
    setShowForm(false);
  };
 
  const handleEdit = (row) => {
	setFormData(row);
    setEditingRow(row.id);
	setShowForm(true);
  };
 
  const handleDelete = (id) => {
    setRows(rows.filter((r) => r.id !== id));
  };
 
  const handleNextSector = (rowIndex) => {
	const updatedRows = [...rows];
	const currentSectorIndex = sectors.indexOf(updatedRows[rowIndex].sector);
	const nextSectorIndex = (currentSectorIndex + 1) % sectors.length;
 
	if (updatedRows[rowIndex].status === "completed") {
  	const newRow = {
        ...updatedRows[rowIndex],
    	id: Date.now(),
    	sector: sectors[nextSectorIndex],
    	startTime: "",
    	endTime: "",
    	status: "pending",
  	};
      updatedRows.splice(rowIndex + 1, 0, newRow);
	} else {
      updatedRows[rowIndex].sector = sectors[nextSectorIndex];
      updatedRows[rowIndex].startTime = "";
      updatedRows[rowIndex].endTime = "";
      updatedRows[rowIndex].status = "pending";
	}
 
    setRows(updatedRows);
  };
 
  const handleStatusChange = (rowIndex, status) => {
	const updatedRows = [...rows];
    updatedRows[rowIndex].status = status;
	if (status === "completed") {
      updatedRows[rowIndex].endTime = new Date().toLocaleTimeString("pt-BR", {
    	hour: "2-digit",
    	minute: "2-digit",
  	});
	} else if (status === "inProgress" && !updatedRows[rowIndex].startTime) {
      updatedRows[rowIndex].startTime = new Date().toLocaleTimeString("pt-BR", {
    	hour: "2-digit",
    	minute: "2-digit",
  	});
	}
    setRows(updatedRows);
  };
 
  const calculateTimeSpent = (startTime, endTime) => {
	if (startTime && endTime) {
  	const start = new Date(`1970-01-01T${startTime}:00`);
  	const end = new Date(`1970-01-01T${endTime}:00`);
  	return Math.round((end - start) / (1000 * 60));
	}
	return "N/A";
  };
 
  const calculateEfficiency = (processedWeight, timeSpent) => {
	if (timeSpent > 0) {
  	return (processedWeight / timeSpent).toFixed(2);
	}
	return "0.00";
  };
 
  const getStatusColor = (status) => {
	switch (status) {
  	case "pending":
    	return "bg-gray-400";
  	case "inProgress":
    	return "bg-yellow-400";
  	case "completed":
    	return "bg-green-400";
  	default:
    	return "";
	}
  };
 
  const filterDataByDateRange = (data) => {
	return data.filter((row) => {
  	const rowDate = new Date(row.date);
  	const start = new Date(dateFilter.startDate);
  	const end = new Date(dateFilter.endDate);
  	return rowDate >= start && rowDate <= end;
	});
  };
 
  const filteredRows = filterDataByDateRange(rows);
 
  const calculateOEE = () => {
	const completedRows = filteredRows.filter(
  	(row) => row.status === "completed"
	);
	if (completedRows.length === 0) return 0;
 
	const totalPlannedTime = completedRows.reduce((acc, row) => {
  	const timeSpent = calculateTimeSpent(row.startTime, row.endTime);
  	return acc + (timeSpent === "N/A" ? 0 : timeSpent);
	}, 0);
