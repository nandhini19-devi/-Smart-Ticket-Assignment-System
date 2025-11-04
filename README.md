import React, { useState, useEffect } from 'react';
import { Users, Clock, AlertCircle, CheckCircle, TrendingUp } from 'lucide-react';

const TicketAssignmentSystem = () => {
  const [tickets, setTickets] = useState([
    { id: 1, title: "Login issues on mobile app", priority: "high", category: "Technical", status: "unassigned", created: new Date(Date.now() - 3600000) },
    { id: 2, title: "Billing inquiry for subscription", priority: "medium", category: "Billing", status: "unassigned", created: new Date(Date.now() - 7200000) },
    { id: 3, title: "Feature request: Dark mode", priority: "low", category: "Feature", status: "unassigned", created: new Date(Date.now() - 1800000) },
    { id: 4, title: "Password reset not working", priority: "high", category: "Technical", status: "unassigned", created: new Date(Date.now() - 900000) },
    { id: 5, title: "Account upgrade question", priority: "medium", category: "Sales", status: "unassigned", created: new Date(Date.now() - 5400000) },
  ]);

  const [agents, setAgents] = useState([
    { id: 1, name: "Sarah Chen", skills: ["Technical", "Feature"], workload: 2, availability: "available" },
    { id: 2, name: "Mike Johnson", skills: ["Billing", "Sales"], workload: 1, availability: "available" },
    { id: 3, name: "Lisa Park", skills: ["Technical", "Billing"], workload: 3, availability: "busy" },
    { id: 4, name: "Tom Wilson", skills: ["Sales", "Feature"], workload: 0, availability: "available" },
  ]);

  const [assignmentRule, setAssignmentRule] = useState("smart");
  const [activityLog, setActivityLog] = useState([]);

  const assignTicket = (ticketId, agentId = null) => {
    const ticket = tickets.find(t => t.id === ticketId);
    let assignedAgent;

    if (agentId) {
      assignedAgent = agents.find(a => a.id === agentId);
    } else {
      assignedAgent = findBestAgent(ticket);
    }

    if (assignedAgent) {
      setTickets(prev => prev.map(t => 
        t.id === ticketId ? { ...t, status: "assigned", assignedTo: assignedAgent.name } : t
      ));
      
      setAgents(prev => prev.map(a => 
        a.id === assignedAgent.id ? { ...a, workload: a.workload + 1 } : a
      ));

      setActivityLog(prev => [{
        time: new Date(),
        message: `Ticket #${ticketId} assigned to ${assignedAgent.name}`,
        type: "assignment"
      }, ...prev.slice(0, 9)]);
    }
  };

  const findBestAgent = (ticket) => {
    const availableAgents = agents.filter(a => 
      a.availability === "available" && a.skills.includes(ticket.category)
    );

    if (availableAgents.length === 0) {
      return agents.filter(a => a.skills.includes(ticket.category))
        .sort((a, b) => a.workload - b.workload)[0];
    }

    switch (assignmentRule) {
      case "smart":
        return availableAgents.sort((a, b) => {
          const priorityWeight = ticket.priority === "high" ? 2 : ticket.priority === "medium" ? 1 : 0.5;
          return (a.workload * priorityWeight) - (b.workload * priorityWeight);
        })[0];
      
      case "roundRobin":
        return availableAgents.sort((a, b) => a.workload - b.workload)[0];
      
      case "skill":
        return availableAgents.filter(a => a.skills[0] === ticket.category)[0] || availableAgents[0];
      
      default:
        return availableAgents[0];
    }
  };

  const autoAssignAll = () => {
    tickets.filter(t => t.status === "unassigned").forEach(ticket => {
      assignTicket(ticket.id);
    });
  };

  const getPriorityColor = (priority) => {
    switch (priority) {
      case "high": return "text-red-600 bg-red-50 border-red-200";
      case "medium": return "text-yellow-600 bg-yellow-50 border-yellow-200";
      case "low": return "text-green-600 bg-green-50 border-green-200";
      default: return "text-gray-600 bg-gray-50 border-gray-200";
    }
  };

  const getTimeAgo = (date) => {
    const minutes = Math.floor((Date.now() - date) / 60000);
    if (minutes < 60) return `${minutes}m ago`;
    const hours = Math.floor(minutes / 60);
    return `${hours}h ago`;
  };

  const unassignedCount = tickets.filter(t => t.status === "unassigned").length;
  const assignedCount = tickets.filter(t => t.status === "assigned").length;

  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50 p-6">
      <div className="max-w-7xl mx-auto">
        <div className="bg-white rounded-xl shadow-lg p-6 mb-6">
          <div className="flex items-center justify-between mb-6">
            <div>
              <h1 className="text-3xl font-bold text-gray-800 flex items-center gap-3">
                <Users className="text-indigo-600" />
                Smart Ticket Assignment System
              </h1>
              <p className="text-gray-600 mt-2">Automated support ticket distribution with intelligent routing</p>
            </div>
            <div className="flex gap-4">
              <div className="text-center bg-indigo-50 px-6 py-3 rounded-lg">
                <div className="text-2xl font-bold text-indigo-600">{unassignedCount}</div>
                <div className="text-sm text-gray-600">Unassigned</div>
              </div>
              <div className="text-center bg-green-50 px-6 py-3 rounded-lg">
                <div className="text-2xl font-bold text-green-600">{assignedCount}</div>
                <div className="text-sm text-gray-600">Assigned</div>
              </div>
            </div>
          </div>

          <div className="flex gap-4 items-center mb-6">
            <label className="text-sm font-medium text-gray-700">Assignment Strategy:</label>
            <select 
              value={assignmentRule}
              onChange={(e) => setAssignmentRule(e.target.value)}
              className="px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
            >
              <option value="smart">Smart (Workload + Priority)</option>
              <option value="roundRobin">Round Robin (Balanced)</option>
              <option value="skill">Skill-Based Priority</option>
            </select>
            <button
              onClick={autoAssignAll}
              disabled={unassignedCount === 0}
              className="ml-auto px-6 py-2 bg-indigo-600 text-white rounded-lg hover:bg-indigo-700 disabled:bg-gray-300 disabled:cursor-not-allowed transition-colors font-medium"
            >
              Auto-Assign All ({unassignedCount})
            </button>
          </div>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <div className="lg:col-span-2 space-y-4">
            <div className="bg-white rounded-xl shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <AlertCircle className="text-indigo-600" size={24} />
                Support Tickets
              </h2>
              <div className="space-y-3">
                {tickets.map(ticket => (
                  <div key={ticket.id} className="border border-gray-200 rounded-lg p-4 hover:shadow-md transition-shadow">
                    <div className="flex items-start justify-between mb-2">
                      <div className="flex-1">
                        <div className="flex items-center gap-3 mb-2">
                          <span className="text-sm font-semibold text-gray-500">#{ticket.id}</span>
                          <span className={`px-3 py-1 rounded-full text-xs font-semibold border ${getPriorityColor(ticket.priority)}`}>
                            {ticket.priority.toUpperCase()}
                          </span>
                          <span className="px-3 py-1 rounded-full text-xs font-medium bg-gray-100 text-gray-700">
                            {ticket.category}
                          </span>
                        </div>
                        <h3 className="font-semibold text-gray-800 mb-1">{ticket.title}</h3>
                        <div className="flex items-center gap-4 text-sm text-gray-500">
                          <span className="flex items-center gap-1">
                            <Clock size={14} />
                            {getTimeAgo(ticket.created)}
                          </span>
                          {ticket.status === "assigned" && (
                            <span className="flex items-center gap-1 text-green-600">
                              <CheckCircle size={14} />
                              Assigned to {ticket.assignedTo}
                            </span>
                          )}
                        </div>
                      </div>
                      {ticket.status === "unassigned" && (
                        <button
                          onClick={() => assignTicket(ticket.id)}
                          className="px-4 py-2 bg-indigo-600 text-white text-sm rounded-lg hover:bg-indigo-700 transition-colors font-medium"
                        >
                          Assign
                        </button>
                      )}
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>

          <div className="space-y-4">
            <div className="bg-white rounded-xl shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <Users className="text-indigo-600" size={24} />
                Support Agents
              </h2>
              <div className="space-y-3">
                {agents.map(agent => (
                  <div key={agent.id} className="border border-gray-200 rounded-lg p-4">
                    <div className="flex items-center justify-between mb-2">
                      <h3 className="font-semibold text-gray-800">{agent.name}</h3>
                      <span className={`px-2 py-1 rounded-full text-xs font-medium ${
                        agent.availability === "available" ? "bg-green-100 text-green-700" : "bg-yellow-100 text-yellow-700"
                      }`}>
                        {agent.availability}
                      </span>
                    </div>
                    <div className="text-sm text-gray-600 mb-2">
                      Skills: {agent.skills.join(", ")}
                    </div>
                    <div className="flex items-center gap-2">
                      <div className="flex-1 bg-gray-200 rounded-full h-2">
                        <div 
                          className="bg-indigo-600 h-2 rounded-full transition-all"
                          style={{ width: `${Math.min(agent.workload * 25, 100)}%` }}
                        />
                      </div>
                      <span className="text-xs font-medium text-gray-600">{agent.workload} tickets</span>
                    </div>
                  </div>
                ))}
              </div>
            </div>

            <div className="bg-white rounded-xl shadow-lg p-6">
              <h2 className="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
                <TrendingUp className="text-indigo-600" size={24} />
                Activity Log
              </h2>
              <div className="space-y-2 max-h-64 overflow-y-auto">
                {activityLog.length === 0 ? (
                  <p className="text-gray-500 text-sm text-center py-4">No assignments yet</p>
                ) : (
                  activityLog.map((log, idx) => (
                    <div key={idx} className="text-sm border-l-2 border-indigo-300 pl-3 py-2">
                      <div className="text-gray-700">{log.message}</div>
                      <div className="text-gray-400 text-xs">{log.time.toLocaleTimeString()}</div>
                    </div>
                  ))
                )}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default TicketAssignmentSystem;
