# OBI

associações de professor, aluno e disciplina.

Chat com base nos models e controllers faça um controller sobre Class e sobre notes

MODELS:
const mongoose = require("mongoose")

const ClassDisciplineSchema = new mongoose.Schema({
   turma_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Class', required: true },
   disciplina_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Discipline', required: true }
});
        
module.exports = mongoose.model('ClassDiscipline', ClassDisciplineSchema);


-------------------------------------------------------------------------------------

const mongoose = require('mongoose')

const ClassSchema = new mongoose.Schema({
    name: { type: String, require: true },
    description: { type: String, require: true },
    author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    dateCreation: { type: Date, default: Date.now } 
})

module.exports = mongoose.model('Class', ClassSchema)

------------------------------------------------------------------------------------

const mongoose = require('mongoose')

const CommunicationSchema = new mongoose.Schema({
   title: { type: String, require: true },
   content: { type: String, require: true },
   author: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
   dateCreation: { type: Date, default: Date.now } 
})

module.exports = mongoose.model('Communication', CommunicationSchema)

------------------------------------------------------------------------------------------

const mongoose = require('mongoose')

const DisciplineSchema = new mongoose.Schema({
    name: { type: String, require: true },
    description: { type: String, require: true }      
})

module.exports = mongoose.model('Discipline', DisciplineSchema)

----------------------------------------------------------------------------------------

const mongoose = require('mongoose');

const noteSchema = new mongoose.Schema({
  aluno_id: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true }, 
  disciplina_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Discipline', required: true }, 
  conceito: { type: String, required: true } 
}, {
  timestamps: true 
});

module.exports = mongoose.model('note', noteSchema);

-----------------------------------------------------------------------------------------------------

const mongoose = require('mongoose');

const ProfessorDisciplineSchema = new mongoose.Schema({
    professor_id: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
        required: true
    },
    disciplina_id: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Discipline',
        required: true
    }
});

module.exports = mongoose.model('ProfessorDiscipline', ProfessorDisciplineSchema);

----------------------------------------------------------------------------------------------------

const mongoose = require("mongoose")

const StudentClassSchema = new mongoose.Schema({
   student_id: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
   class_id: { type: mongoose.Schema.Types.ObjectId, ref: 'Class', required: true }
});

module.exports = mongoose.model('StudentClass', StudentClassSchema);

---------------------------------------------------------------------------------------------------

const mongoose = require('mongoose')
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
   username: {type: String, required: true, unique: true},
   useremail: {type: String, required: true, unique: true},
   password: { type: String, required: true },
   usertype: { type: String, enum: ['Aluno', 'Professor', 'Coordenacao'], required: true }       
})

UserSchema.pre('save', async function (next) {
    if (this.isModified('password')) {
       this.password = await bcrypt.hash(this.password, 10)   
    }      

    next()
})

UserSchema.methods.comparePassword = function(password) {
   return bcrypt.compare(password, this.password)       
}

module.exports = mongoose.model("User", UserSchema)


CONTROLLERS:

const Comunicado = require('../models/communicationsModel');
const User = require('../models/userModel');

// Cria um novo comunicado
exports.createComunicado = async (req, res) => {
  try {
    const comunicado = new Comunicado(req.body);
    await comunicado.save();
    res.status(201).json(comunicado);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

// Obtém todos os comunicados
exports.getComunicados = async (req, res) => {
  try {
    const comunicados = await Comunicado.find().populate('author', 'username');
    res.json(comunicados);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Obtém comunicado por ID
exports.getComunicadoById = async (req, res) => {
  try {
    const comunicado = await Comunicado.findById(req.params.id).populate('author', 'username'); 
    if (!comunicado) {
      return res.status(404).json({ message: 'Comunicado não encontrado' });
    }
    res.json(comunicado);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Atualiza comunicado por ID
exports.updateComunicado = async (req, res) => {
  try {
    const comunicado = await Comunicado.findByIdAndUpdate(req.params.id, req.body, { new: true });
    if (!comunicado) {
      return res.status(404).json({ message: 'Comunicado não encontrado' });
    }
    res.json(comunicado);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Deleta comunicado por ID
exports.deleteComunicado = async (req, res) => {
  try {
    await Comunicado.findByIdAndDelete(req.params.id);
    res.json({ message: 'Comunicado deletado' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Obtém comunicados por usuário (autor)
exports.getComunicadosByUserId = async (req, res) => {
  try {
    const { userId } = req.params;

    const user = await User.findById(userId);
    if (!user) {
      return res.status(404).json({ message: 'Usuário não encontrado' });
    }

    const comunicados = await Comunicado.find({ author: userId });

    res.status(200).json(comunicados);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};


-----------------------------------------------------------------------------------------------

const Disciplina = require("../models/disciplineModel");
const User = require("../models/userModel"); 
const ProfessorDiscipline = require("../models/professorDisciplineModel"); 

// Criação de disciplina e associação com professor
exports.createDisciplina = async (req, res) => {
  try {
    const { name, description, professorId } = req.body;

    // Cria a nova disciplina
    const disciplina = new Disciplina({ name, description });
    await disciplina.save();

    // Verifica se o professor existe
    const professor = await User.findById(professorId);
    if (!professor || professor.usertype !== "Professor") {
      return res.status(400).json({ message: 'Professor inválido ou não encontrado' });
    }

    // Cria a associação entre o professor e a disciplina
    const professorDiscipline = new ProfessorDiscipline({
      professor_id: professorId,
      disciplina_id: disciplina._id
    });
    await professorDiscipline.save();

    res.status(201).json({ disciplina, professorDiscipline });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

// Buscar todas as disciplinas, incluindo o professor associado
exports.getDisciplinas = async (req, res) => {
  try {
    const disciplinas = await Disciplina.find().populate({
      path: 'ProfessorDiscipline',
      populate: {
        path: 'professor_id',
        select: 'username useremail'
      }
    });

    res.json(disciplinas);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Buscar disciplina por ID, incluindo o professor associado
exports.getDisciplinaById = async (req, res) => {
  try {
    const disciplina = await Disciplina.findById(req.params.id).populate({
      path: 'ProfessorDiscipline',
      populate: {
        path: 'professor_id',
        select: 'username useremail'
      }
    });

    if (!disciplina) {
      return res.status(404).json({ message: 'Disciplina não encontrada' });
    }

    res.json(disciplina);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Atualização da disciplina
exports.updateDisciplina = async (req, res) => {
  try {
    const disciplina = await Disciplina.findByIdAndUpdate(req.params.id, req.body, { new: true });

    if (!disciplina) {
      return res.status(404).json({ message: 'Disciplina não encontrada' });
    }

    res.json(disciplina);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Deletar disciplina e sua associação com o professor
exports.deleteDisciplina = async (req, res) => {
  try {
    await Disciplina.findByIdAndDelete(req.params.id);

    // Remove a associação com o professor
    await ProfessorDiscipline.deleteMany({ disciplina_id: req.params.id });

    res.json({ message: 'Disciplina deletada e associações removidas' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};


------------------------------------------------------------------------------------------------

const User = require('../models/userModel')

exports.getUsers = async (req, res) => {
   try {
     const users = await User.find()
     res.json(users)     
   } catch (error) {
      res.status(500).json({error: error.message})    
   }       
}

exports.getUserById = async (req, res) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    res.json(user);
 } catch (error) {
   res.status(500).json({ error: error.message });
  }
};


exports.updateUser = async (req, res) => {
  try {
    const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
    if (!user) {
      return res.status(404).json({ message: 'User not found' });
    }
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
        

exports.deleteUser = async (req, res) => {
  try {
     await User.findByIdAndDelete(req.params.id);
     res.json({ message: 'User deleted' });
 }   catch (error) {
    res.status(500).json({ error: error.message });
  }
};
