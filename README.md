import React, { useState } from 'react';
import { View, Text, StyleSheet, TouchableOpacity, FlatList, TextInput } from 'react-native';

export default function App() {

  // ================= STATE =================
  const [level, setLevel] = useState(1);
  const [xp, setXP] = useState(0);
  const [quests, setQuests] = useState([]);
  const [aiMessage, setAiMessage] = useState("SYSTEM INITIALIZING...");
  const [input, setInput] = useState("");

  const xpRequired = level * 50;

  // ================= RANK SYSTEM =================
  const getRank = () => {
    if (level <= 5) return "E";
    if (level <= 10) return "D";
    if (level <= 15) return "C";
    if (level <= 20) return "B";
    if (level <= 25) return "A";
    return "S";
  };

  // ================= GEMINI AI =================
  const callGemini = async (lvl, xpVal) => {
    try {
      const response = await fetch(
        "https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=AIzaSyBLcMoYg9DZNbGOExpAvi12nqAuAeX3_8g",
        {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            contents: [
              {
                parts: [
                  {
                    text: `You are SYSTEM ARISE. User Level: ${lvl}, XP: ${xpVal}. Respond in ONE short strict line, cold and intimidating.`
                  }
                ]
              }
            ]
          })
        }
      );

      const data = await response.json();
      return data?.candidates?.[0]?.content?.parts?.[0]?.text || "SYSTEM ERROR";

    } catch (e) {
      return "CONNECTION FAILURE";
    }
  };

  // ================= ADD QUEST =================
  const addQuest = () => {
    if (!input.trim()) return;

    const newQuest = {
      id: Date.now().toString(),
      title: input,
      xp: 20,
      completed: false
    };

    setQuests([...quests, newQuest]);
    setInput("");
  };

  // ================= COMPLETE QUEST =================
  const completeQuest = async (quest) => {
    let newXP = xp + quest.xp;
    let newLevel = level;

    while (newXP >= newLevel * 50) {
      newXP -= newLevel * 50;
      newLevel++;
    }

    setXP(newXP);
    setLevel(newLevel);

    setQuests(quests.map(q =>
      q.id === quest.id ? { ...q, completed: true } : q
    ));

    const msg = await callGemini(newLevel, newXP);
    setAiMessage(msg);
  };

  // ================= UI =================
  return (
    <View style={styles.container}>

      {/* SYSTEM PANEL */}
      <View style={styles.systemBox}>
        <Text style={styles.systemTitle}>SYSTEM STATUS</Text>
        <Text style={styles.systemText}>{aiMessage}</Text>
      </View>

      {/* PLAYER STATS */}
      <View style={styles.center}>
        <Text style={styles.level}>LEVEL {level}</Text>
        <Text style={styles.rank}>RANK {getRank()}</Text>
        <Text style={styles.xp}>{xp} / {xpRequired} XP</Text>
      </View>

      {/* ADD QUEST */}
      <View style={styles.inputRow}>
        <TextInput
          style={styles.input}
          placeholder="Enter quest..."
          placeholderTextColor="#555"
          value={input}
          onChangeText={setInput}
        />
        <TouchableOpacity style={styles.addBtn} onPress={addQuest}>
          <Text style={styles.addText}>ADD</Text>
        </TouchableOpacity>
      </View>

      {/* QUEST LIST */}
      <FlatList
        data={quests}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <TouchableOpacity
            style={[styles.quest, item.completed && { opacity: 0.4 }]}
            onPress={() => completeQuest(item)}
          >
            <Text style={styles.questText}>
              {item.title} (+{item.xp} XP)
            </Text>
          </TouchableOpacity>
        )}
      />

    </View>
  );
}

// ================= STYLES =================
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#000",
    padding: 20
  },

  systemBox: {
    borderWidth: 1,
    borderColor: "#00F0FF",
    padding: 15,
    borderRadius: 12,
    marginBottom: 20
  },

  systemTitle: {
    color: "#00F0FF",
    fontSize: 16,
    fontWeight: "bold"
  },

  systemText: {
    color: "#007BFF",
    marginTop: 6
  },

  center: {
    alignItems: "center",
    marginBottom: 20
  },

  level: {
    color: "#00F0FF",
    fontSize: 36,
    fontWeight: "bold"
  },

  rank: {
    color: "#8A2BE2",
    fontSize: 20
  },

  xp: {
    color: "#007BFF"
  },

  inputRow: {
    flexDirection: "row",
    marginBottom: 20
  },

  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: "#00F0FF",
    borderRadius: 10,
    padding: 10,
    color: "#fff"
  },

  addBtn: {
    marginLeft: 10,
    borderWidth: 1,
    borderColor: "#00F0FF",
    padding: 10,
    borderRadius: 10,
    justifyContent: "center"
  },

  addText: {
    color: "#00F0FF"
  },

  quest: {
    padding: 15,
    borderWidth: 1,
    borderColor: "#007BFF",
    borderRadius: 10,
    marginBottom: 10
  },

  questText: {
    color: "#fff"
  }
});
