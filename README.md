import React, { useState, useRef, useEffect } from 'react';
import { Send, Phone, Video, MoreVertical, Search, Paperclip, Mic, Smile, Image, FileText, Camera, Plus, X } from 'lucide-react';

const WhatsAppClone = () => {
  const [selectedChat, setSelectedChat] = useState(0);
  const [message, setMessage] = useState('');
  const [showAttachMenu, setShowAttachMenu] = useState(false);
  const [showReactionPicker, setShowReactionPicker] = useState(null);
  const [newGroupMode, setNewGroupMode] = useState(false);
  const [groupName, setGroupName] = useState('');
  const [selectedContacts, setSelectedContacts] = useState([]);
  
  const reactions = ['❤️', '👍', '😂', '😮', '😢', '😡'];
  
  const [chats, setChats] = useState([
    {
      id: 1,
      name: 'John Doe',
      avatar: '👨‍💼',
      lastMessage: 'Hey! How are you doing?',
      time: '10:30 AM',
      unread: 2,
      isGroup: false,
      messages: [
        { id: 1, text: 'Hey there!', sent: false, time: '10:28 AM', reactions: {} },
        { id: 2, text: 'Hi John! How are you?', sent: true, time: '10:29 AM', reactions: {} },
        { id: 3, text: 'Hey! How are you doing?', sent: false, time: '10:30 AM', reactions: {} }
      ]
    },
    {
      id: 2,
      name: 'Sarah Wilson',
      avatar: '👩‍💻',
      lastMessage: 'See you tomorrow!',
      time: '9:15 AM',
      unread: 0,
      isGroup: false,
      messages: [
        { id: 1, text: 'Are we still on for the meeting?', sent: true, time: '9:10 AM', reactions: {} },
        { id: 2, text: 'Yes, absolutely! See you at 2 PM', sent: false, time: '9:12 AM', reactions: {} },
        { id: 3, text: 'Perfect! See you tomorrow!', sent: false, time: '9:15 AM', reactions: {} }
      ]
    },
    {
      id: 3,
      name: 'Team Group',
      avatar: '👥',
      lastMessage: 'Thanks everyone!',
      time: '8:45 AM',
      unread: 5,
      isGroup: true,
      participants: ['You', 'Alice', 'Bob', 'Charlie'],
      messages: [
        { id: 1, text: 'Good morning team!', sent: false, time: '8:30 AM', sender: 'Alice', reactions: {} },
        { id: 2, text: 'Morning! Ready for today\'s sprint', sent: true, time: '8:35 AM', reactions: {} },
        { id: 3, text: 'Let\'s make it a great day!', sent: false, time: '8:40 AM', sender: 'Bob', reactions: {} },
        { id: 4, text: 'Thanks everyone!', sent: false, time: '8:45 AM', sender: 'Charlie', reactions: {} }
      ]
    }
  ]);

  const [availableContacts] = useState([
    { id: 4, name: 'Alice Smith', avatar: '👩‍🎨' },
    { id: 5, name: 'Bob Johnson', avatar: '👨‍🔬' },
    { id: 6, name: 'Charlie Brown', avatar: '👨‍🎓' },
    { id: 7, name: 'Diana Prince', avatar: '👩‍⚕️' },
    { id: 8, name: 'Eve Davis', avatar: '👩‍🚀' }
  ]);
  
  const messagesEndRef = useRef(null);
  const fileInputRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(() => {
    scrollToBottom();
  }, [chats]);

  const sendMessage = (messageText = message, fileData = null) => {
    if (messageText.trim() || fileData) {
      const newMessage = {
        id: Date.now(),
        text: messageText,
        sent: true,
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        reactions: {},
        file: fileData
      };
      
      setChats(prev => prev.map(chat => 
        chat.id === chats[selectedChat].id 
          ? { 
              ...chat, 
              messages: [...chat.messages, newMessage],
              lastMessage: fileData ? `📎 ${fileData.name}` : messageText,
              time: newMessage.time
            }
          : chat
      ));
      
      setMessage('');
      setShowAttachMenu(false);
      
      // Simulate response after 1-2 seconds
      setTimeout(() => {
        const responses = [
          "That's interesting!",
          "Got it, thanks!",
          "Sounds good to me",
          "I'll check on that",
          "Perfect! 👍",
          "Nice share!",
          "Thanks for sending that!"
        ];
        
        const response = {
          id: Date.now(),
          text: responses[Math.floor(Math.random() * responses.length)],
          sent: false,
          time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
          reactions: {},
          sender: chats[selectedChat].isGroup ? chats[selectedChat].participants[Math.floor(Math.random() * (chats[selectedChat].participants.length - 1)) + 1] : null
        };
        
        setChats(prev => prev.map(chat => 
          chat.id === chats[selectedChat].id 
            ? { 
                ...chat, 
                messages: [...chat.messages, response],
                lastMessage: response.text,
                time: response.time
              }
            : chat
        ));
      }, 1000 + Math.random() * 1000);
    }
  };

  const handleFileSelect = (e) => {
    const file = e.target.files[0];
    if (file) {
      const fileData = {
        name: file.name,
        size: file.size,
        type: file.type,
        url: URL.createObjectURL(file)
      };
      sendMessage(`Shared a file: ${file.name}`, fileData);
    }
  };

  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      sendMessage();
    }
  };

  const addReaction = (messageId, reaction) => {
    setChats(prev => prev.map(chat => 
      chat.id === chats[selectedChat].id 
        ? {
            ...chat,
            messages: chat.messages.map(msg => 
              msg.id === messageId 
                ? {
                    ...msg,
                    reactions: {
                      ...msg.reactions,
                      [reaction]: (msg.reactions[reaction] || 0) + 1
                    }
                  }
                : msg
            )
          }
        : chat
    ));
    setShowReactionPicker(null);
  };

  const createGroup = () => {
    if (groupName.trim() && selectedContacts.length > 0) {
      const newGroup = {
        id: Date.now(),
        name: groupName,
        avatar: '👥',
        lastMessage: 'Group created',
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        unread: 0,
        isGroup: true,
        participants: ['You', ...selectedContacts.map(c => c.name)],
        messages: [{
          id: 1,
          text: 'Group created',
          sent: false,
          time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
          reactions: {},
          isSystem: true
        }]
      };
      
      setChats(prev => [newGroup, ...prev]);
      setNewGroupMode(false);
      setGroupName('');
      setSelectedContacts([]);
      setSelectedChat(0);
    }
  };

  const toggleContactSelection = (contact) => {
    setSelectedContacts(prev => 
      prev.find(c => c.id === contact.id)
        ? prev.filter(c => c.id !== contact.id)
        : [...prev, contact]
    );
  };

  if (newGroupMode) {
    return (
      <div className="flex h-screen bg-gray-100">
        <div className="w-full bg-white flex flex-col">
          {/* Header */}
          <div className="bg-green-600 text-white p-4 flex items-center justify-between">
            <div className="flex items-center space-x-3">
              <button onClick={() => setNewGroupMode(false)} className="p-1 hover:bg-green-700 rounded">
                <X size={20} />
              </button>
              <h1 className="text-xl font-semibold">New Group</h1>
            </div>
            <button 
              onClick={createGroup}
              disabled={!groupName.trim() || selectedContacts.length === 0}
              className="px-4 py-2 bg-green-500 hover:bg-green-400 disabled:bg-green-800 disabled:opacity-50 rounded-lg"
            >
              Create
            </button>
          </div>

          {/* Group Name Input */}
          <div className="p-4 border-b border-gray-200">
            <input
              type="text"
              value={groupName}
              onChange={(e) => setGroupName(e.target.value)}
              placeholder="Group name"
              className="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-green-500"
            />
          </div>

          {/* Selected Contacts */}
          {selectedContacts.length > 0 && (
            <div className="p-4 border-b border-gray-200">
              <h3 className="text-sm font-semibold text-gray-600 mb-2">Selected ({selectedContacts.length})</h3>
              <div className="flex flex-wrap gap-2">
                {selectedContacts.map(contact => (
                  <div key={contact.id} className="flex items-center space-x-2 bg-green-100 rounded-full px-3 py-1">
                    <span className="text-sm">{contact.avatar} {contact.name}</span>
                    <button 
                      onClick={() => toggleContactSelection(contact)}
                      className="text-green-600 hover:text-green-800"
                    >
                      <X size={14} />
                    </button>
                  </div>
                ))}
              </div>
            </div>
          )}

          {/* Contact List */}
          <div className="flex-1 overflow-y-auto">
            <div className="p-4">
              <h3 className="text-sm font-semibold text-gray-600 mb-4">Select Contacts</h3>
              {availableContacts.map(contact => (
                <div 
                  key={contact.id}
                  onClick={() => toggleContactSelection(contact)}
                  className="flex items-center space-x-3 p-3 hover:bg-gray-50 rounded-lg cursor-pointer"
                >
                  <div className="w-10 h-10 bg-gray-300 rounded-full flex items-center justify-center text-xl">
                    {contact.avatar}
                  </div>
                  <div className="flex-1">
                    <h4 className="font-medium">{contact.name}</h4>
                  </div>
                  <div className={`w-5 h-5 rounded-full border-2 ${
                    selectedContacts.find(c => c.id === contact.id) 
                      ? 'bg-green-500 border-green-500' 
                      : 'border-gray-300'
                  }`}>
                    {selectedContacts.find(c => c.id === contact.id) && (
                      <div className="w-full h-full flex items-center justify-center text-white text-xs">✓</div>
                    )}
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="flex h-screen bg-gray-100">
      {/* Sidebar */}
      <div className="w-1/3 bg-white border-r border-gray-200 flex flex-col">
        {/* Header */}
        <div className="bg-gray-50 p-4 border-b border-gray-200">
          <div className="flex items-center justify-between">
            <h1 className="text-xl font-semibold text-gray-800">WhatsApp</h1>
            <div className="flex space-x-2">
              <button 
                onClick={() => setNewGroupMode(true)}
                className="p-2 text-gray-600 hover:bg-gray-200 rounded-full"
                title="New Group"
              >
                <Plus size={20} />
              </button>
              <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
                <Search size={20} />
              </button>
              <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
                <MoreVertical size={20} />
              </button>
            </div>
          </div>
        </div>

        {/* Search */}
        <div className="p-3 border-b border-gray-200">
          <div className="relative">
            <Search className="absolute left-3 top-3 text-gray-400" size={18} />
            <input
              type="text"
              placeholder="Search or start new chat"
              className="w-full pl-10 pr-4 py-2 bg-gray-100 rounded-lg focus:outline-none focus:bg-white focus:ring-2 focus:ring-green-500"
            />
          </div>
        </div>

        {/* Chat List */}
        <div className="flex-1 overflow-y-auto">
          {chats.map((chat, index) => (
            <div
              key={chat.id}
              onClick={() => setSelectedChat(index)}
              className={`p-4 border-b border-gray-100 cursor-pointer hover:bg-gray-50 transition-colors ${
                selectedChat === index ? 'bg-green-50' : ''
              }`}
            >
              <div className="flex items-center space-x-3">
                <div className="w-12 h-12 bg-gray-300 rounded-full flex items-center justify-center text-2xl">
                  {chat.avatar}
                </div>
                <div className="flex-1 min-w-0">
                  <div className="flex items-center justify-between">
                    <h3 className="font-semibold text-gray-900 truncate">
                      {chat.name} {chat.isGroup && <span className="text-xs text-gray-500">({chat.participants.length})</span>}
                    </h3>
                    <span className="text-xs text-gray-500">{chat.time}</span>
                  </div>
                  <div className="flex items-center justify-between mt-1">
                    <p className="text-sm text-gray-600 truncate">{chat.lastMessage}</p>
                    {chat.unread > 0 && (
                      <span className="bg-green-500 text-white text-xs rounded-full px-2 py-1 min-w-[20px] text-center">
                        {chat.unread}
                      </span>
                    )}
                  </div>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>

      {/* Chat Area */}
      <div className="flex-1 flex flex-col">
        {/* Chat Header */}
        <div className="bg-gray-50 p-4 border-b border-gray-200 flex items-center justify-between">
          <div className="flex items-center space-x-3">
            <div className="w-10 h-10 bg-gray-300 rounded-full flex items-center justify-center text-xl">
              {chats[selectedChat].avatar}
            </div>
            <div>
              <h2 className="font-semibold text-gray-900">{chats[selectedChat].name}</h2>
              <p className="text-xs text-gray-500">
                {chats[selectedChat].isGroup 
                  ? `${chats[selectedChat].participants.join(', ')}`
                  : 'Online'
                }
              </p>
            </div>
          </div>
          <div className="flex space-x-2">
            <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
              <Phone size={20} />
            </button>
            <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
              <Video size={20} />
            </button>
            <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
              <MoreVertical size={20} />
            </button>
          </div>
        </div>

        {/* Messages */}
        <div className="flex-1 overflow-y-auto p-4 bg-gradient-to-b from-green-50 to-green-100">
          {chats[selectedChat].messages.map((msg) => (
            <div
              key={msg.id}
              className={`mb-4 flex ${msg.sent ? 'justify-end' : 'justify-start'}`}
            >
              <div className={`max-w-xs lg:max-w-md ${msg.sent ? 'mr-2' : 'ml-2'}`}>
                {msg.isSystem ? (
                  <div className="text-center text-gray-500 text-sm py-2">
                    {msg.text}
                  </div>
                ) : (
                  <div
                    className={`px-4 py-2 rounded-lg relative group ${
                      msg.sent
                        ? 'bg-green-500 text-white'
                        : 'bg-white text-gray-800 shadow-sm'
                    }`}
                  >
                    {msg.sender && !msg.sent && (
                      <p className="text-xs font-semibold text-green-600 mb-1">{msg.sender}</p>
                    )}
                    
                    {msg.file && (
                      <div className="mb-2 p-2 bg-gray-100 rounded-lg">
                        <div className="flex items-center space-x-2">
                          {msg.file.type.startsWith('image/') ? (
                            <Image size={16} className="text-gray-600" />
                          ) : (
                            <FileText size={16} className="text-gray-600" />
                          )}
                          <span className="text-sm text-gray-600">{msg.file.name}</span>
                        </div>
                        {msg.file.type.startsWith('image/') && (
                          <img src={msg.file.url} alt={msg.file.name} className="mt-2 max-w-full h-32 object-cover rounded" />
                        )}
                      </div>
                    )}
                    
                    <p className="text-sm">{msg.text}</p>
                    <p className={`text-xs mt-1 ${msg.sent ? 'text-green-100' : 'text-gray-500'}`}>
                      {msg.time}
                    </p>
                    
                    {Object.keys(msg.reactions).length > 0 && (
                      <div className="flex flex-wrap gap-1 mt-2">
                        {Object.entries(msg.reactions).map(([reaction, count]) => (
                          <span key={reaction} className="text-xs bg-gray-200 px-2 py-1 rounded-full">
                            {reaction} {count}
                          </span>
                        ))}
                      </div>
                    )}
                    
                    {/* Reaction Button */}
                    <button
                      onClick={() => setShowReactionPicker(showReactionPicker === msg.id ? null : msg.id)}
                      className="absolute -top-2 -right-2 opacity-0 group-hover:opacity-100 bg-gray-600 text-white p-1 rounded-full transition-opacity"
                    >
                      <Smile size={12} />
                    </button>
                    
                    {/* Reaction Picker */}
                    {showReactionPicker === msg.id && (
                      <div className="absolute top-8 right-0 bg-white border rounded-lg shadow-lg p-2 flex space-x-1 z-10">
                        {reactions.map(reaction => (
                          <button
                            key={reaction}
                            onClick={() => addReaction(msg.id, reaction)}
                            className="hover:bg-gray-100 p-1 rounded"
                          >
                            {reaction}
                          </button>
                        ))}
                      </div>
                    )}
                  </div>
                )}
              </div>
            </div>
          ))}
          <div ref={messagesEndRef} />
        </div>

        {/* Message Input */}
        <div className="bg-gray-50 p-4 border-t border-gray-200">
          <div className="flex items-center space-x-2">
            <div className="relative">
              <button 
                onClick={() => setShowAttachMenu(!showAttachMenu)}
                className="p-2 text-gray-600 hover:bg-gray-200 rounded-full"
              >
                <Paperclip size={20} />
              </button>
              
              {/* Attachment Menu */}
              {showAttachMenu && (
                <div className="absolute bottom-12 left-0 bg-white border rounded-lg shadow-lg p-2 space-y-1">
                  <button 
                    onClick={() => fileInputRef.current?.click()}
                    className="flex items-center space-x-2 w-full p-2 hover:bg-gray-100 rounded"
                  >
                    <FileText size={16} />
                    <span className="text-sm">Document</span>
                  </button>
                  <button 
                    onClick={() => fileInputRef.current?.click()}
                    className="flex items-center space-x-2 w-full p-2 hover:bg-gray-100 rounded"
                  >
                    <Image size={16} />
                    <span className="text-sm">Photo</span>
                  </button>
                  <button 
                    onClick={() => fileInputRef.current?.click()}
                    className="flex items-center space-x-2 w-full p-2 hover:bg-gray-100 rounded"
                  >
                    <Camera size={16} />
                    <span className="text-sm">Camera</span>
                  </button>
                </div>
              )}
            </div>
            
            <div className="flex-1 relative">
              <input
                type="text"
                value={message}
                onChange={(e) => setMessage(e.target.value)}
                onKeyPress={handleKeyPress}
                placeholder="Type a message..."
                className="w-full px-4 py-2 pr-12 border border-gray-300 rounded-full focus:outline-none focus:ring-2 focus:ring-green-500"
              />
              <button className="absolute right-3 top-2 p-1 text-gray-600 hover:bg-gray-200 rounded-full">
                <Smile size={18} />
              </button>
            </div>
            
            {message.trim() ? (
              <button
                onClick={() => sendMessage()}
                className="p-2 bg-green-500 text-white rounded-full hover:bg-green-600 transition-colors"
              >
                <Send size={20} />
              </button>
            ) : (
              <button className="p-2 text-gray-600 hover:bg-gray-200 rounded-full">
                <Mic size={20} />
              </button>
            )}
          </div>
        </div>
      </div>
      
      {/* Hidden File Input */}
      <input
        ref={fileInputRef}
        type="file"
        onChange={handleFileSelect}
        className="hidden"
        accept="image/*,application/pdf,.doc,.docx,.txt"
      />
    </div>
  );
};

export default WhatsAppClone;
